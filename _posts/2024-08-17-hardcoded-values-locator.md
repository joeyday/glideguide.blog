---
layout: post
title: Hardcoded Values Locator
author: Joey
date: 2024-08-17
categories:
  - script include
  - workflow
  - flow designer
---

<span class="lead">I don't know about your ServiceNow environment</span>, but in our environment we have a lot of technical debt built up in the form of hardcoded values (users, groups, etc.) in legacy Workflows and Flow Designer Flows.

For example, we might have hardcoded some specific user to be an approver on a request, but of course it's only a matter of time before that user transfers departments, gets a promotion, or leaves the company and then that process is broken. In an effort to mitigate the issue, we might use a group instead, but if nobody's paying attention it doesn't take long for all the users in the group to rotate out and then we're right back in the same boat.

Though it probably deserves a blog post all its own, preventing/solving this problem by establishing better practices up front isn't my topic today. Instead I want to share a project I worked on recently that's enabled us to be more proactive in reducing incidents related to the problem.

## The Script Include

Here's a Script Include that can help you locate hardcoded values in legacy Workflows and Flow Designer Flows:

~~~ javascript
var HardcodedValuesLocator = function (table, tableQuery) {
	return (function() {

		/////// INITIALIZATION ///////

		if (typeof tableQuery === 'undefined')
			tableQuery = new GlideQuery();

		var displayColumn = gs.getDisplayColumn(table);

		var records = new GlideQuery(table)
			.where(tableQuery)
			.whereNotNull(displayColumn)
			.select(displayColumn)
			.reduce(function (acc, record) {
				acc[record.sys_id] = record[displayColumn];
				return acc;
			}, {});

		var recordIDs = Object.keys(records);

		/////// PUBLIC FUNCTIONS ///////
		
		function findInFlows(flowQuery) {
			flowQuery = flowQuery || new GlideQuery()
				.where('active', true);

			var flows = new GlideQuery('sys_hub_flow')
				.where(flowQuery)
				.select()
				.reduce(function (acc, flow) {
					return acc.concat(flow.sys_id);
				}, []);

			var actions = new GlideQuery('sys_hub_action_instance')
				.where('flow', 'IN', flows)
				.select('flow', 'flow$DISPLAY')
				.reduce(function (acc, action) {
					acc[action.sys_id] = action;
					return acc;
				}, {});

			var result = new GlideQuery('sys_element_mapping')
				.where('id', 'IN', Object.keys(actions))
				.whereNotNull('value')
				.select('value', 'id')
				.map(function (mapping) {
					mapping.values = mapping.value.split(/\b/); // split on word boundaries
					return mapping;
				})
				.filter(function (mapping) {
					return mapping.values.some(function (value) {
						return Boolean(records[value]);
					});
				})
				.reduce(function (acc, mapping) {
					return acc.concat(mapping);
				}, []);

			result = result.concat(new GlideQuery('sys_variable_value')
				.where('document', 'sys_hub_action_instance')
				.where('document_key', 'IN', Object.keys(actions))
				.where('variable.reference', table)
				.whereNotNull('value')
				.select('value', 'document_key')
				.map(function (variable) {
					variable.id = variable.document_key;
					variable.values = variable.value.split(',');
					return variable;
				})
				.filter(function (variable) {
					return variable.values.some(function (value) {
						return Boolean(records[value]);
					});
				})
				.reduce(function (acc, variable) {
					return acc.concat(variable);
				}, []);

			function getDisplayValuesWithReferences() {
				return result.reduce(function (acc, mapping) {
					var flow = actions[mapping.id].flow$DISPLAY.trim();
					if (typeof acc[flow] === 'undefined') {
						acc[flow] = [];
					}

					var intersection = mapping.values
						.reduce(function (acc, value) {
							var record = records[value];
							if (record) acc.push(record.trim());
							return acc;
						}, []);

					acc[flow] = acc[flow].concat(intersection);
					return acc;
				}, {});
			}
			
			function getSysIds() {
				return result.reduce(function (acc, mapping) {
					var flowId = actions[mapping.id].flow;
					if (acc.indexOf(flowId) == -1)
						acc.push(flowId);
					return acc;
				}, []);
			}
			
			return {
				getDisplayValuesWithReferences: getDisplayValuesWithReferences,
				getSysIds: getSysIds
			};
		}

		function findInWorkflows(workflowVersionQuery) {
			workflowVersionQuery = workflowVersionQuery || new GlideQuery()
					.where('published', true);

			var workflowVersions = new GlideQuery('wf_workflow_version')
				.where(workflowVersionQuery)
				.select()
				.reduce(function (acc, workflow) {
					return acc.concat(workflow.sys_id);
				}, []);

			var activities = new GlideQuery('wf_activity')
				.where('workflow_version', 'IN', workflowVersions)
				.select('workflow_version', 'workflow_version$DISPLAY')
				.reduce(function (acc, activity) {
					acc[activity.sys_id] = activity;
					return acc;
				}, {});

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
			
			function getDisplayValuesWithReferences() {
				return result.reduce(function (acc, variable) {
					var workflow = activities[variable.document_key].workflow_version$DISPLAY.trim();
					if (typeof acc[workflow] === 'undefined') {
						acc[workflow] = [];
					}

					var intersection = variable.values
						.reduce(function (acc, value) {
							var record = records[value];
							if (record) acc.push(record);
							return acc;
						}, []);

					acc[workflow] = acc[workflow].concat(intersection);
					return acc;
				}, {});
			}
			
			function getSysIds() {
				return result.reduce(function (acc, variable) {
					var workflowId = activities[variable.document_key].workflow_version;
					if (acc.indexOf(workflowId) == -1)
						acc.push(workflowId);
					return acc;
				}, []);
			}
			
			return {
				getDisplayValuesWithReferences: getDisplayValuesWithReferences,
				getSysIds: getSysIds
			};
		}

		/////// RETURN ///////

		return {
			findInFlows: findInFlows,
			findInWorkflows: findInWorkflows
		};
	
	})();
};
~~~

### Basic usage

So far we've been using this by just calling it manually in a background script and having it log out the results, but you could conceivably call it from a scheduled job to either notify someone or raise tasks to clean up newly discovered issues. Even better might be calling it from an ATF Test.

When you call the HardcodedValuesLocator you must provide a table you're looking for hardcoded values from (`sys_user` and `sys_user_group` are the common ones, but go nuts if you have something else you're looking for). You get back an object with a couple of methods, `findInFlows` and `findInWorkflows`, which do pretty much what they say on the tin. You can of course assign the object to a variable and then make the subsequent method call like this:

~~~ javascript
var hvl = HardcodedValuesLocator('sys_user');
var result = hvl.findInFlows();
~~~

Or I like to chain the method calls together more fluidly like this:

~~~ javascript
var result = HardcodedValuesLocator('sys_user')
	.findInFlows();
~~~

The result here is another object that has the data you're after, but not yet in a form that's immediately useful. You have two options at this step, (1) get the data as sys_ids by calling the `getSysIds` method. This would be useful if you're building some automated process and you need the result to be machine-readable so you can perform subsequent processing yourself.

~~~ javascript
var machineReadableResult = HardcodedValuesLocator('sys_user')
	.findInFlows()
	.getSysIds();
	
// Your processing logic goes here...
~~~

Or (2) get the data in human-readable display values with the `getDisplayValuesWithReferences` method, and here you'll probably want to send the result to the System Log:

~~~ javascript
var humanReadableResult = HardcodedValuesLocator('sys_user')
  .findInFlows()
  .getDisplayValuesWithReferences();
  
gs.info(JSON.stringify(humanReadableResult, null, 2);
~~~

## Advanced options

There are a few more parameters you can pass to these functions to make your searches more targeted. First, when you call the HardcodedValuesLocator Script Include you can optionally pass a GlideQuery object as the second parameter. This is useful if you're only looking for specific users, like inactive ones:

~~~ javascript
HardcodedValuesLocator('sys_user', new GlideQuery().where('active', false);
~~~

Additionally, you can filter which Workflows you're searching by optionally passing a GlideQuery object to the `findInFlows` method, for example, if you only want to search in currently published Flows:

~~~ javascript
hvl.findInFlows(new GlideQuery().where('active', true));
~~~

Your use cases for this will likely vary from mine, so you're welcome to call these methods with whatever queries you need, but here's an example script I've been using to categorize and sort hardcoded users and groups by most concerning to least concerning:

~~~ javascript
(function () {
	// Get some useful data up front
	var catalogWorkflows = new GlideQuery('sc_cat_item')
		.where('active', true)
		.whereNotNull('workflow')
		.select('workflow')
		.reduce(function (acc, catItem) {
			return acc.concat(catItem.workflow);
		}, []);

	var catalogWorkflowVersions = new GlideQuery()
		.where('published', true)
		.where('active', true)
		.where('workflow', 'IN', catalogWorkflows);

	var otherWorkflowVersions = new GlideQuery()
		.where('published', true)
		.where('active', true)
		.where('table', '!=', 'sc_req_item');

	var groups = new GlideQuery('sys_user_grmember')
		.whereNotNull('user')
		.whereNotNull('group')
		.groupBy('group')
		.aggregate('count');

	var groupsWithActiveMembers = groups
		.where('user.active', true)
		.select()
		.reduce(function (acc, groupMember) {
			return acc.concat(groupMember.group.group);
		}, []);

	var groupsWithInactiveMembers = groups
		.where('user.active', false)
		.select()
		.reduce(function (acc, groupMember) {
			return acc.concat(groupMember.group.group);
		}, []);

	// Define searches we want to perform from most to least concerning
	var searches = [
		{
			description: 'Hardcoded inactive users',
			table: 'sys_user',
			query: new GlideQuery()
				.where('active', false)
		},
		{
			description: 'Hardcoded groups with no active members',
			table: 'sys_user_group',
			query: new GlideQuery()
				.where('sys_id', 'NOT IN', groupsWithActiveMembers)
		},
		{
			description: 'Hardcoded groups with at least one inactive member',
			table: 'sys_user_group',
			query: new GlideQuery()
				.where('sys_id', 'IN', groupsWithInactiveMembers)
		},
		{
			description: 'Hardcoded active users',
			table: 'sys_user',
			query: new GlideQuery()
				.where('active', true)
		}
	];

	for (var i = 0; i < searches.length; i++) {
		// Do each search in all three places and output the results
		var flowsResult = HardcodedValuesLocator(searches[i].table, searches[i].query)
			.findInFlows(new GlideQuery().where('active', true))
			.getDisplayValuesWithReferences();
		
		if (Object.keys(flowsResult).length) {
			gs.print('\n' + searches[i].description + ' in flows:');
			gs.print(flowsResult);
		}

		var catalogWorkflowsResult = HardcodedValuesLocator(searches[i].table, searches[i].query)
			.findInWorkflows(catalogWorkflowVersions)
			.getDisplayValuesWithReferences();
		
		if (Object.keys(catalogWorkflowsResult).length) {
			gs.print('\n' + searches[i].description + ' in catalog workflows:');
			gs.print(flowsResult);
		}

		var otherWorkflowsResult = HardcodedValuesLocator(searches[i].table, searches[i].query)
			.findInWorkflows(otherWorkflowVersions)
			.getDisplayValuesWithReferences();
		
		if (Object.keys(otherWorkflowsResult).length) {
			gs.print('\n' + searches[i].description + ' in other workflows:');
			gs.print(flowsResult);
		}
	}
})();
~~~

Note I'm using `gs.print()` in the above sample, which works fine 

## Conclusion

I hope this helps you find and clean up some of the skeletons in your ServiceNow closet. If you do find it useful or have any questions, feel free to reach out on [SNDevs Slack](https://invite.sndevs.com/) or [Mastodon](https://social.sndevs.com/).{% include endmark.html %}
