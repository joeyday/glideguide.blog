---
layout: post
title: Scripted Due Dates in the Ask For Approval Action
author: Dan
date: 2023-05-10
categories: Flow Designer
---
Something in Flow Designer that has been bothering me for several months, if not a year or two, is that I've been unable to get scripted due dates to work in the Ask For Approval action. If you're unfamiliar, due dates provide the ability to specify how long the flow should wait for approval, as well as what action should be taken on that approval when the specified due date arrives. This prevents a Flow from waiting endlessly for an approval response. 

The due date input allows for scripted input to be entered and toggled on/off, but until today I thought the scriptable input option was entirely broken. I've written many unsuccessful due date scripts that return date/time strings, GlideDateTime objects, numeric duration values, and others.

Why is it sometimes necessary to script a due date value anyway? Let's say for example that you wanted an approval to automatically cancel if left unresponded after 14 calendar days. At first glance this seems entirely possible using the relative due date configuration options, until you get to the "From" date/time value input that must be provided. How can you specify that you want the timer to start at the time of Approval record creation? You can't at least not without writing a script. As with all other inputs in Flow Designer it would like you to drag and drop a date/time pill value and move on. If this Flow is based on a Catalog Item it's tempting to drag and drop the Created On date/time pill of the RITM value, and in some cases (depending on where this Action is in your Flow) this may not be much of a problem. One way you might solve this problem is to create a Flow Variable of type String and then populate that variable with the desired date/time value using the *Set Flow Variables* Action. While I dislike this solution for a number of reasons it does work. What doesn't sit right with me is that the scripted option is available and the only reason it doesn't work is because ServiceNow doesn't provide any helpful information on how to use it in the [Ask For Approval Action documention](https://docs.servicenow.com/bundle/utah-build-workflows/page/administer/flow-designer/reference/ask-approval-flow-designer.html).

I recently happened to see that the due date configuration is visible when viewing a Flow Execution Context. After finding this I immediately made some assumptions on other possible values and began to test my theories. The JSON object below is the result of my testing. I have documented each property's purpose and possible values. 

**Due Date Configuration Template**
~~~ json
{
    "action": "", // approve, cancel, reject
    "date_type": "", // actual, relative
    "date": "", // Date/Time string used as the static date/time value, or as the starting date/time value for relative due dates.
    // The following can be ignored for static due date configurations
    "duration": 1, // The duration value (e.g. 1 minute, 1 hour, 1 day, etc.). ** Default value is 1 **
    "duration_type": "days", // minutes, hours, days, weeks, months, quarters, years. ** Default value is days ** 
    "schedule": "", // Sys ID of the Schedule [cmn_schedule] record to be used when calculating relative due dates
    "schedule_label": "" // Label of the Schedule [cmn_schedule] record to be used when calculating relative due dates
}
~~~

**Scripted Due Date Example**
~~~ javascript
var gdt = new GlideDateTime();
gdt.addDaysLocalTime(14);

var config = {
  "action": "cancel", // approve, cancel, reject
  "date_type": "actual", // actual, relative
  "date": gdt.getValue(), // date/time string
}

return JSON.stringify(config);
~~~

{% include endmark.html %}

