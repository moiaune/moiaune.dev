---
title: "ServiceNow Impersonate Dialog Bookmark"
description: "A bookmark for opening the impersonate dialog on any ServiceNow instance"
tags: ['servicenow', 'javascript']
date: 2025-02-11T16:30:00+01:00
draft: false
---

If you didnt know ServiceNow has a page called `impersonate_dialog.do` where you can choose who to impersonate. Ofcourse, inside the ServiceNow application you can always click on your profile image and choose "Impersonate user" (granted you have the neccessary role). But sometimes you need to impersonate a user that might only have access to the Service Portal. Unless you are using the `/esc` portal, you might not have the "End Impersonation" option in the menu.

So here's an easy trick to always be able to end or impersonate another user.

In your browser, add a new bookmark and give it a describing name e.g "Impersonation Dialog". Then put the following as the URL:

```
javascript:window.location.pathname="/impersonate_dialog.do"
```

This snippet will use the current domain and add "/impersonate_dialog.do" to the end of it. So no matter which instance you are on, it will always work. No need for a seperate bookmark for each instance.
