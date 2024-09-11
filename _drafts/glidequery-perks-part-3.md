---
layout: post
title: 'GlideQuery Perks, Part 3: Fun, Expressive Results Processing'
author: Joey
date: 2023-01-01
categories:
 - glidequery
 - glidequery perks series
---

One of the problems I never knew GlideRecord has is there's only a couple ways to get records out of it, either `get()` for a single record or `query()` and `next()` for multiple records. GlideQuery breaks the mold by offering a gaggle of new and expressive ways to loop through the resulting records.

An early draft of this entry basically just regurgitated ServiceNow's own documentation about these features, but I want to provide more value than that by showing more examples, design patterns, and best practices for how to use the various methods proficiently.

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

{% include endmark.html %}

