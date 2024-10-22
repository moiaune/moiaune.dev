---
title: "Working with comments and work notes in ServiceNow"
description: "Additional comments and Work notes is of type Journal List and therefor we can not get their value directly. So here's how to interact with comments and work_notes."
tags: ['servicenow', 'javascript']
date: 2021-08-17T10:13:04+02:00
draft: false
---

Additional comments and Work notes is of type `Journal List` and therefor we can not get their value directly. So here's how to interact with `comments` and `work_notes`.

<!--more-->

```javascript
(function(current) {

    // Get the latest entry
    var lastComment = current.comments.getJournalEntry(1);
    var lastWorkNote = current.work_notes.getJournalEntry(1);

    // Get all entries
    var allComments = current.comments.getJournalEntry(-1);
    var allWorkNotes = current.work_notes.getJournalEntry(-1);

})(current);
```

## Official documentation

- [getJournalEntry(Number mostRecent) | GlideElement | ServiceNow Developers](https://developer.servicenow.com/dev.do#!/reference/api/quebec/server/no-namespace/c_GlideElementScopedAPI#SGE-getJournalEntry_N)
- [GlideElement | ServiceNow Developers](https://developer.servicenow.com/dev.do#!/reference/api/quebec/server/no-namespace/c_GlideElementScopedAPI)
