---
layout: post
title: "Friends Don't Let Friends Use GlideAjax"
author: Joey
date: 2023-02-02
categories: 
- GlideaAax
- XMLHttpRequest
- REST
- Fetch
---

<span class="lead">I used to hate writing [GlideAjax](https://docs.servicenow.com/bundle/tokyo-application-development/page/app-store/dev_portal/API_reference/GlideAjax/concept/c_GlideAjaxAPI.html) Script Includes</span>, but I haven't written one in probably five years. How have I avoided it? Easy! Let me show you how to hit the [Table <abbr>API</abbr>](https://docs.servicenow.com/bundle/tokyo-application-development/page/integrate/inbound-rest/concept/c_TableAPI.html) in a Client Script using vanilla JavaScript.

The preferred way using modern JavaScript is with the [Fetch <abbr>API</abbr>](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API). Here's an example to pull the ten oldest Incident records:

~~~ javascript
var table = 'incident';
var params = [
  'sysparm_query=active=true^ORDERBYsys_created_on',
  'sysparm_fields=number,short_description,description',
  'sysparm_limit=10',
  // 'sysparm_display_value=all', // true for display values, all for both
];

var endpoint = '/api/now/table/' + table + '?' + params.join('&');

fetch(endpoint, { headers: { 'X-UserToken': g_ck } }) // user auth token
  .then(function (response) {
    if (!response.ok)
      throw 'HTTP status ' + response.status + ' on ' + endpoint;
      
    return response.json();
  })
  .then(function (response) {
    // Do stuff (e.g. number will be response.result.number)
    // ...
  });
~~~

I know what you're gonna say, this looks like more code than a typical GlideAjax call. That actually depends on how many parameters you use and whether you need error handling, but keep in mind this is _all_ the code. There's no need for you to make any server-side component because the Table <abbr>API</abbr> does all the heavy lifting for you. One Client Script and you're done.

Note, the `g_ck` variable is a client-side variable that stores the logged-in user's session token. It's the easiest way to authenticate the current user to the Table <abbr>API</abbr> so you don't have to pass along any other credential. Be aware <abbr>ACL</abbr>s will be honored, so you may need to find another way to do this if your Client Script needs to fetch records the user can't otherwise read.

Also note, the Fetch <abbr>API</abbr> has [pretty broad support](https://caniuse.com/?search=fetch) (and don't forget this is running in your end users' browsers, so you're not stuck on <abbr>ES</abbr>5 like you are most everywhere else in ServiceNow), but if your environment won't allow it for whatever reason (say your company mandates Internet Explorer), here's the same example using bog standard [<abbr>XML</abbr>HttpRequest](https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest) instead:

~~~ javascript
var table = 'incident';
var params = [
  'sysparm_query=active=true^ORDERBYsys_created_on',
  'sysparm_fields=number,short_description,description',
  'sysparm_limit=10',
  // 'sysparm_display_value=all', // true for display values, all for both
];

var endpoint = '/api/now/table/' + table + '?' + params.join('&');

var xhr = new XMLHttpRequest();
xhr.open('GET', encodeURI(endpoint));
xhr.setRequestHeader('X-UserToken', g_ck); // user auth token
xhr.send();

xhr.onreadystatechange = function () {
  if (xhr.readyState !== XMLHttpRequest.DONE)
    return;
    
  if (xhr.status < 200 || xhr.status >= 300)
    throw 'HTTP status ' + xhr.status + ' on ' + endpoint;

  var response = JSON.parse(xhr.responseText);
  
  // Do stuff (e.g. number will be response.result.number)
  // ...
};
~~~

Of course, you could refactor either of these examples to use any endpoint you like. Maybe you really need to crunch some data server-side and deliver it to the client in a way the Table <abbr>API</abbr> just can't accommodate. You could create your own [Scripted <abbr>REST</abbr> <abbr>API</abbr>](https://docs.servicenow.com/bundle/tokyo-application-development/page/integrate/custom-web-services/concept/c_CustomWebServices.html) endpoint and ping that from the client instead. The world is your oyster, but please, and I say this as a friend, never use GlideAjax again. You're welcome!{% include endmark.html %}