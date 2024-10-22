---
title: "Copy to Clipboard in Servicenow"
description: "A simple snippet for copying fields or other values to clipboard from a UI Action in ServiceNow"
tags: ["ServiceNow", "Javascript"]
date: 2024-03-22T14:10:35+01:00
draft: false
---

Recently I was asked to find a way for users to easily create a string consisting of the task number and short description that they could paste into the time management software. In this short article we're going to take a look at how we can copy something to your system clipboard from a UI Action in ServiceNow.

<!--more-->

## The snippet

```javascript
if (navigator.clipboard.writeText) {
    var number = g_form.getValue('number');
    var short_description = g_form.getValue('short_description');
    navigator.clipboard.writeText(number + " - " + short_description).then(function() {
        console.log("Copied!");
    });
}
```

The `if` condition check wether your browser supports this function. Then we retrieve the value of fields `number` and `short_description` using `g_form`. Lastly we asynchronously write the the string to system clipboard. The `navigator.clipboard.writeText` is a native Javascript function and can be used on other sites aswell.
