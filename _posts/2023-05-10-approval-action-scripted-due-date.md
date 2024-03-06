---
layout: post
title: Scripted Due Dates in the Ask For Approval Action
author: Dan
date: 2023-05-10
categories:
   - flow designer
---

<span class="lead">Something in Flow Designer</span> that has always bothered me is that I've been unable to get scripted due dates to work in the Ask For Approval action. If you're unfamiliar, due dates provide the ability to specify how long the flow should wait for approval, as well as what action should be taken on the approval when the specified due date arrives. This prevents a flow from waiting endlessly for an approval response. 

It is sometimes necessary to script the due date value in the Ask For Approval action. For example, if you wanted an approval to automatically cancel when left unresponded after 14 calendar days. At first glance this seems entirely possible using the relative due date configuration options, until you get to the "From" date/time value input that must be provided. How can you specify that you want the timer to start at the time of Approval record creation? To my knowledge you cannot, at least not without writing a script. As with all other inputs in Flow Designer it would like you to drag and drop a date/time pill value and move on. If this flow is based on a Catalog Item it can be tempting to drag and drop the Created On date/time pill from the RITM, and in some cases (depending on where this Action is in your Flow) this may not be much of a problem. 

<img style="width: 90%; display: block !important; margin: auto;" src="/assets/images/2023-05-10-ask-for-approval-action.png" alt="Ask For Approval Action" />

One workaround that I have used after my failed attempts at using the script input has been to populate a flow variable with the desired date/time string value. I can populate the variable via script using the Set Flow Variables action. While I dislike this solution for a number of reasons it does work. 

<img style="width: 90%; display: block !important; margin: auto;" src="/assets/images/2023-05-10-variable-workaround.png" alt="Set Flow Variable action" />
<img style="width: 90%; display: block !important; margin: auto;" src="/assets/images/2023-05-10-workaround-pill.png" alt="Add Variable Pill" />

I was recently building a flow using this workaround when I happened to see that a due date configured with pills generates a JSON configuration that you can view from the Flow Execution Context. 

<img style="width: 90%; display: block !important; margin: auto;" src="/assets/images/2023-05-10-execution-context.png" alt="Execution Context" />

After finding this I immediately made some assumptions on other possible values and began to test my theories. The JSON object below is the result of my testing. I have documented each property's purpose and possible values along with an example script. 


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


<img style="width: 90%; display: block !important; margin: auto;" src="/assets/images/2023-05-10-script-example.png" alt="Example Script Input" />


~~~ javascript
var gdt = new GlideDateTime();
gdt.addDaysLocalTime(14);
var config = {
    "action": "cancel", // approve, cancel, reject
    "date_type": "actual", // actual, relative
    "date": gdt.getValue(), // Date/Time string used as the static date/time value, or as the starting date/time value for relative due dates.
    // The following can be ignored for static due date configurations
    "duration": 1, // The duration value (e.g. 1 minute, 1 hour, 1 day, etc.). ** Default value is 1 **
    "duration_type": "days", // minutes, hours, days, weeks, months, quarters, years. ** Default value is days ** 
    "schedule": "", // Sys ID of the Schedule [cmn_schedule] record to be used when calculating relative due dates
    "schedule_label": "" // Label of the Schedule [cmn_schedule] record to be used when calculating relative due dates
}
return JSON.stringify(config);
~~~


While I am delighted to have solved this riddle, I can't emphasize enough that to my knowledge all of this is completely undocumented, at least as of the [Utah release](https://docs.servicenow.com/bundle/utah-build-workflows/page/administer/flow-designer/reference/ask-approval-flow-designer.html), and is likely unrecommended as any instances of this action using a scripted input are subject to break in the event that the action is updated. I cannot guarantee that this will work for you or that it will continue to work. Now that I have seen how the action works I suspect that in the event you find yourself in a scenario like the one described above, the intended solution would be to use a flow variable or similar steps of scripting a dynamic date/time and provide that value via pill.{% include endmark.html %}
