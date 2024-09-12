---
layout: post
title: 'GlideQuery Perks, Part 3: Fun, Expressive Results Processing'
author: Joey
date: 2023-01-01
categories:
 - glidequery
 - glidequery perks series
---

// In case I get lost, I left off on the MACUtility Script Include (12 occurrences found)
// sys_script_include_42e5bbf8878fa150eae6fd94dabb352c.xml

One of the problems I never knew GlideRecord has is there's only a couple ways to get records out of it, either `get()` for a single record or `query()` and `next()` for multiple records. GlideQuery breaks the mold by offering a gaggle of new and expressive ways to loop through the resulting records.

An early draft of this entry basically just regurgitated ServiceNow's own documentation about these features, but I want to provide more value than that by showing more examples, design patterns, and best practices for how to use the various methods proficiently.

## Common use cases
- Get a single field value from the database
- Get a single record from the database with multiple fields
- Get several records and load them into an array
- Get several records and load them into an object keyed by some unique field
- Get several records and load them into an array, sorted somehow
- Get several records and load them into an array, filtered somehow
- De-duplicate an array (either using groupBy/aggregate or filter)
- Insert a record
- Update a record
- Update multiple records
- Get a record if it exists, then upsert it
- Get a record if it exists, then upsert it, and also keep it in a variable for easy reference after the fact
- Get aggregate data (count, avg, min, max)
- Get records matching some criteria from table x, but the records should be related to
  some records from another table y (a lot of people solve this with sub-queries leading  
  to N+1 complexity. I do do separate queries, one to load records from y and cache them
  in some sort of data structure, then two to load records from x.)


## select vs. selectOne

~~~ javascript
return new GlideQuery('u_oncall_autoassignment_rule')
  .orderBy('sys_created_on')
  .where('u_group', group)
  .where('u_task_table', taskTable)
  .where('u_active', true)
  .selectOne('u_task_condition');
~~~

## map, filter, and reduce

~~~ javascript
var records = new GlideQuery(table)
  .where(tableQuery)
  .whereNotNull(displayColumn)
  .select(displayColumn)
  .reduce(function (acc, record) {
    acc[record.sys_id] = record[displayColumn];
    return acc;
  }, {});
~~~

~~~ javascript
var workflowVersions = new GlideQuery('wf_workflow_version')
  .where(workflowVersionQuery)
  .select()
  .reduce(function (acc, workflow) {
    return acc.concat(workflow.sys_id);
  }, []);
~~~

~~~ javascript
var activities = new GlideQuery('wf_activity')
  .where('workflow_version', 'IN', workflowVersions)
  .select('workflow_version', 'workflow_version$DISPLAY')
  .reduce(function (acc, activity) {
    acc[activity.sys_id] = activity;
    return acc;
  }, {});
~~~

~~~ javascript
var result = new GlideQuery('sys_variable_value')
  .where('document', 'wf_activity')
  .where('document_key', 'IN', Object.keys(activities))
  .where('variable.reference', table)
  .whereNotNull('value')
  .select('value', 'document_key')
  .map(function (variable) {
    variable.values = variable.value.split(',');
    return variable;
  })
  .filter(function (variable) {
    return variable.values.some(function (value) {
      return Boolean(records[value]);
    });
  });
~~~

~~~ javascript
return new GlideQuery('u_oncall_autoassignment_rule')
  .select('u_group')
  .reduce(function (arr, rule) {
    return arr.concat(rule.u_group);
  }, []);
~~~

~~~ javascript
return new GlideQuery('cmn_rota')
  .where('active', true)
  .select('group')
  .reduce(function (arr, rota) {
    // de-dupe the array while building
    if (arr.indexOf(rota.group) == -1) arr.push(rota.group);
    return arr;
  }, []);
~~~

~~~ javascript
// This object tracks which memberships are in our table but are no
// longer found in Slack so we can remove them from our side
var notFoundMemberships = new global.GlideQuery('x_prole_slack_int_channel_membership')
  .whereNotNull('channel')
  .whereNotNull('user')
  .select('channel.id', 'user.u_slack_id')
  .reduce(function (acc, membership) {

    // If we haven't seen this channel yet,
    // make a new empty object for it
    if (typeof acc[membership.channel.id] === 'undefined') {
      acc[membership.channel.id] = {};
    }
    
    // Add this user along with the sys_id of the
    // membership record to the channel object
    acc[membership.channel.id][membership.user.u_slack_id] = membership.sys_id;
    
    return acc;

  }, {});
~~~

~~~ javascript
return new global.GlideQuery(table)
  .whereNotNull(mapField)
  .select(mapField)
  .reduce(function (records, record) {
    records[record[mapField]] = record.sys_id;
    return records;
  }, {});
~~~

~~~ javascript
// Example of de-duping values
// Could've alternatively used a Set
// Or maybe best would've been aggregate and groupBy?
var seenTesters = {};

var testers = new global.GlideQuery('rm_scrum_task')
  .where('story', current.getValue('story'))
  .where('type', '4')
  .select('assigned_to$DISPLAY')
  .map(function (task) {
    return task.assigned_to$DISPLAY;
  })
  .filter(function(assignee) {
    if (seenTesters.hasOwnProperty(assignee)) return false;
    seenTesters[assignee] = true;
    return true;
  })
  .reduce(function (acc, assignee) {
    return acc.concat(assignee);
  }, []);
~~~

~~~ javascript
var userData = new GlideQuery('x_prole_tp_onboarding')
  .where('start_date', new GlideDate())
  .select('user', 'username', 'first_name', 'last_name', 'manager', 'job_title$DISPLAY', 'department', 'location')
  .reduce(function(acc, record) {
    if (record.user !== null) {
      acc[record.user] = {
        user_name: record.username,
        first_name: record.first_name,
        last_name: record.last_name,
        email: record.username + '@progleasing.com',
        manager: record.manager,
        // u_job_title: record.job_title, // Once the sys_user table is referencing the new Position [sn_hr_core_position] table we can use this line
        department: record.department,
        location: record.location,
        company: record.company,
      };
    }
    // Convert the job title to the legacy job title
    var jobTitle = new JobRecordLookup().getLegacyJobTitle(record.job_title$DISPLAY);
    if (jobTitle) {
      acc[record.user].u_job_title = jobTitle;
    }
    return acc;
  }, {});
~~~

~~~ javascript
// The last reduce here could be a map, right?
return new global.GlideQuery('x_prole_ts_code_review')
	.where('engineer', 'IN', engineers)
	.where('state', '!=', 7)
	.aggregate('max', 'sys_created_on')
	.groupBy('engineer')
	.select()
	.reduce(function(acc, review) {
		return acc.concat({
			engineer: review.group.engineer,
			mostRecentDate: review.max.sys_created_on
		});
	}, [])
	.sort(function(a, b) {
		if (a.mostRecentDate < b.mostRecentDate) return -1;
		if (a.mostRecentDate > b.mostRecentDate) return 1;

		// Flip a coin if timestamp is identical
		if (Math.floor(Math.random() * 2)) return -1;
		return 1;
	})
	.reduce(function(acc, review) {
		return acc.concat(review.engineer);
	}, []);
~~~

~~~ javascript
var story_states = new global.GlideQuery('sys_choice')
	.where('name', 'rm_story')
	.where('element', 'state')
	.where('inactive', false)
	.select('value')
	.reduce(function(acc, record) {
		acc[record.sys_id] = Number(record.value);
		return acc;
	}, {});
~~~

~~~ javascript
var schedules = new global.GlideQuery('x_prole_ts_code_review_schedule')
	.select('reviewers', 'groups', 'trainees', 'quantity', 'days_ago', 'states', 'monday', 'tuesday', 'wednesday', 'thursday', 'friday', 'saturday', 'sunday')
	.map(function(record) {
		record.reviewers = (record.reviewers || '').split(',');
		record.groups = (record.groups || '').split(',');
		record.trainees = (record.trainees || '').split(',');
		record.states = (record.states || '').split(',');
		record.states = record.states.map(function(state_id) {
			return story_states[state_id];
		});
		record.days_ago = gs.daysAgoStart(record.days_ago);
		return record;
	})
	.reduce(function(acc, record) {
		weekdays.forEach(function(weekday) {
			if (record[weekday]) {
				if (typeof acc[weekday_map[weekday]] === 'undefined') {
					acc[weekday_map[weekday]] = [];
				}
				acc[weekday_map[weekday]].push(record);
			}
		});
		return acc;
	}, {});
~~~

~~~ javascript
// Choose a random record in a data set
var records = global.GlideQuery.parse(inputs.table, inputs.query)
	.select()
	.reduce(function(acc, record) {
		return acc.concat(record.sys_id);
	}, []);

var rando = Math.floor(Math.random() * records.length);

outputs.random_record_query = 'sys_id=' + records[rando];
~~~

~~~ javascript
// Something more clever going on here to de-dupe records.
// I think I might've found this on StackOverflow.
let approvers = new global.GlideQuery('sys_user')
	.where('sys_id', 'IN', loan_officers)
	.select('location.u_level_2_approvers')
	.map(user => user.location.u_level_2_approvers)
	.reduce(global.AGQ.concat, [])
	.filter((approver, i, approvers) => approvers.indexOf(approver) === i);
~~~

~~~ javascript
const tasks = new global.GlideQuery('x_amr_m_mac_task')
	.where('parent', requestID)
	.where('sys_id', '!=', current.getValue('table_sys_id'))
	.select()
	.map(global.GQ.get('sys_id'))
	.reduce(global.AGQ.push, []);

new global.GlideQuery('question_answer')
	.where('table_name', 'x_amr_m_mac_task')
	.where('table_sys_id', 'IN', tasks)
	.where('question', current.getValue('question'))
	.disableWorkflow()
	.updateMultiple({
		value: current.getValue('value')
	});
~~~

## some and every

## forEach

Best when you need to do some other action for each record, not best for building data structures or processing data down to a single answer.

~~~ javascript
new global.GlideQuery('sys_flow_context')
  .where('state', 'WAITING')
  .select()
  .forEach(function (context) {
    sn_fd.FlowAPI.nudgeFlow(context.sys_id, 1);
  });
~~~

~~~ javascript
new GlideQuery('u_group_has_role_subprod')
  .where('u_instance', instance)
  .select('u_role', 'u_group')
  .forEach(function (record) {
    util.grantRoleToGroup(record.u_role, record.u_group);
  });
~~~

## isPresent and ifPresent

~~~ javascript
var groupHasRole = new GlideQuery('sys_group_has_role')
  .where('role', roleID)
  .where('group', groupID)
  .selectOne()
  .isPresent();
~~~

~~~ javascript
// Check if membership already exists in ServiceNow
var isPresent = new global.GlideQuery('x_prole_slack_int_channel_membership')
  .where('channel', channel.sys_id)
  .where('user', userIDs[member])
  .selectOne()
  .isPresent();

// If not, insert it
if (!isPresent) {
  new global.GlideQuery('x_prole_slack_int_channel_membership')
    .insert({
      channel: channel.sys_id,
      user: userIDs[member]
    });
}
~~~

~~~ javascript
new global.GlideQuery('sys_user')
  .where('u_adp_position_id', supervisorID)
  .selectOne()
  .ifPresent(function(user) {
    current.setValue('manager', user.sys_id);
  });
~~~

## orElse and get

~~~ javascript
// Is this really a good pattern though, or an anti-pattern?
// I've grown to not like methods or logic that potentially
// returns two completely different data types, in this case
// either an object or a boolean.
var story = new GlideQuery('rm_story')
  .where('sys_id', storyID)
  .selectOne('u_effort')
  .orElse(false);

if (!story) return false; // Something's wrong here
~~~

~~~ javascript
// This seems like a better pattern than the one above
var teamRecord = new global.GlideQuery('x_prole_slack_int_slack_workspaces')
  .where('id', team.id)
  .selectOne()
  .orElse({
    id: team.id
  });

// Set values for the workspace
teamRecord.workspace = team.name;
teamRecord.url = team.team_url;

// Insert/update the workspace
new global.GlideQuery('x_prole_slack_int_slack_workspaces')
  .insertOrUpdate(teamRecord);
~~~

~~~ javascript
var batch = new global.GlideQuery('x_prole_tp_onboarding_batch')
  .where('start_date', startDate.getValue())
  .selectOne()
  .orElse({
    start_date: startDate.getValue()
  });

if (typeof batch.sys_id === 'undefined') {
  batch = new global.GlideQuery('x_prole_tp_onboarding_batch')
    .insert(batch)
    .get();
}
~~~

~~~ javascript 
// When would this ever return false and/or when would it log the error?
// Is this a good pattern or a smell?
try {
	var titleQuery = new global.GlideQuery('hr_position')
		.insert({
			'position': jobTitle,
			'active': true
		})
		.get();

	if (titleQuery) {
		return titleQuery.sys_id;
	} else {
		return false;
	}
} catch (ex) {
	gs.error('Error in the createLegacyJobTitle method of the JobRecordLookup SI for jobTitle: *' + jobTitle + '*');
	return false;
}
~~~

~~~ javascript
return new global.GlideQuery('x_prole_tp_onboarding_task')
	.insert({
		onboarding: onboardingGR.getUniqueValue(),
		short_description: short_desc,
		description: desc,
		assignment_group: 'dc8d7eba1339fb40d9ff30ded144b096', // Service Desk Leadership
		type: 'failed_automation',
		process: 'job_record_identification',
		priority: 2,
	}).get();
~~~

## toGlideRecord

~~~ javascript
var gr = new global.GlideQuery('x_amr_el_velocify_lead')
	.where('state', 'IN', [1, 2])
	.where('sys_created_on', '>', gs.daysAgo(daysAgoStart))
	.where('sys_created_on', '<', gs.daysAgo(daysAgoEnd))
	.toGlideRecord();

gr.query();

while (gr.next()) {
	gs.eventQueue('x_amr_el.lead.daysaged.' + daysAgo, gr);
}
~~~

{% include endmark.html %}

