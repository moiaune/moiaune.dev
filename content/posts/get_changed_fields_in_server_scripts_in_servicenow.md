---
title: "Get changed fields in server scripts in ServiceNow"
description: "In ServiceNow you often write Business Rules or some other logic, based on fields that has been changed/updated. For the most part, this can be done via GUI, but sometimes you have to resort to some scripting."
date: 2021-08-17T10:04:06+02:00
tags: ['servicenow', 'javascript']
draft: false
---

In ServiceNow you often write Business Rules or some other logic, based on fields that has been changed/updated. For the most part, this can be done via GUI, but sometimes you have to resort to some scripting. If you ever need to get which fields has been changed/updated, e.g in an advanced filter, this is how you check for it.

```javascript
(function(current){
    var gru = GlideScriptRecordUtil.get(current);

    // Returns an arrayList of changed field elements with friendly names
    var changedFields = gru.getChangedFields();

    //Returns an arrayList of changed field elements with database names
    var changedFieldNames = gru.getChangedFieldNames();

    //Returns an arrayList of all change values from changed fields
    var changes = gru.getChanges();

    // Convert to JavaScript Arrays
    gs.include('j2js');
    changedFields = j2js(changedFields);
    changedFieldNames = j2js(changedFieldNames);
    changeds = j2js(changes);

    gs.info("Changed Fields: " + JSON.stringify(changedFields));
    gs.info("Changed Field Names: " + JSON.stringify(changedFieldNames));
    gs.info("Changes: " + JSON.stringify(changes));
})(current);
```
