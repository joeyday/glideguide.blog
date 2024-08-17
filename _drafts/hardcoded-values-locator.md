---
layout: post
title: Hardcoded Values Locator
author: Joey
date: 2024-04-18
categories:
  - script include
  - workflow
  - flow designer
---

<span class="lead">I don't know about your ServiceNow environment</span>, but in our environment we have a lot of technical debt built up in the form of hardcoded values (users, groups, etc.) in legacy Workflows and Flow Designer Flows.

For example, we might have hardcoded a specific user to be the approver for a certain request, but of course it's only a matter of time before that user wins the lottery and leaves the company, and then the process behind that request is just broken. In an effort to mitigate the issue, we might use a group instead, but if nobody's paying attention it doesn't take long for all the users in the group to rotate out to new positions and then we're right back in the same boat.

Though it probably deserves a blog post all its own, preventing/solving this problem by establishing better practices up front isn't my topic today. Instead I want to share a project I worked on recently which enabled us to be more proactive in mitigating incidents related to the problem.

## The Script Include

Here's a Script Include that can help you locate hardcoded values in legacy Workflows and Flow Designer Flows:

~~~ javascript
var ProgHardcodedReferenceLookup = function (table, tableQuery) {
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
				.reduce(GQX.toArray, []);

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
				.reduce(GQX.toArray, []));

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
			dangerousFindInFlows: dangerousFindInFlows,
			findInWorkflows: findInWorkflows
		};
	
	})();
};
~~~

### How to use it

So far I've been using this by just calling it manually in a background script and having it log out the results, but you could conceivably call it from a scheduled job to either notify someone or raise tasks to clean up newly discovered issues. Even better might be calling it from an ATF Test.

When you instantiate the HardcodedValuesLocator you get back an object with a couple of methods. You can of course assign that object to some variable and then make subsequent method calls like this:

~~~ javascript
ProgHardcodedReferenceLookup(q[i].table, q[i].query)
  .findInWorkflows(catalogWorkflowVersions)
  .getDisplayValuesWithReferences()
~~~


{% include endmark.html %}
