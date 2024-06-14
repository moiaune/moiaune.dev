---
title: "Get Array of Unique Objects in Servicenow"
description: "How to get only unique objects from an array of objects in ServiceNow"
date: 2024-06-14T12:29:18+02:00
tags: ['servicenow', 'javascript']
---

There's no secret that the Javascript support (serverside) in ServiecNow is lacking. And today I came across another little quirk. In ServiceNow we have the `ArrayUtil.unique()` to get an Array of unique values, but this does not support objects. Neither does ServiceNow support `Map()` or `Set()` on the serverside, so here's a little snippet to filter an array of objects and receive unique objects based on a object `key`.

```javascript
/**
 * Get unique objects from an array of object, by key.
 *
 * @param {Array.<Object>} arr - The array of objects
 * @param {string} key - The object key that must be unique
 * @returns {Array.<Object>} An array of unique objects
 */
function uniqueObjects(arr, key) {
    return arr.filter(function(value, index, self) {
        return self.map(function(x) {
            return x[key];
        }).indexOf(value[key]) == index;
    });
}
```
