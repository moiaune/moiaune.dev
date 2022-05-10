---
title: 'Compare two dates in ServiceNow'
date: 2022-05-10T00:00:00+00:00
draft: false
---

To work with date and datetime in ServiceNow we can use the [GlideDateTime API](https://developer.servicenow.com/dev.do#!/reference/api/sandiego/server/no-namespace/c_APIRef).

## Get duration

```javascript
var date1 = new GlideDateTime('2022-05-10 09:00:00');
var date2 = new GlideDateTime('2022-05-12 12:00:00');

var diff = GlideDateTime.subtract(date1, date2);
gs.info(diff.getDisplayValue());

// should print: 2 Days 3 Hours
```

## Adding/removing

```javascript
var date1;

// Adding days
date1 = new GlideDateTime('2022-05-10 09:00:00');
date1.addDaysUTC(2); // 2022-05-12 09:00:00

// Subtract days
date1 = new GlideDateTime('2022-05-10 09:00:00');
date1.addDaysUTC(-2); // 2022-05-08 09:00:00

// Add seconds
date1 = new GlideDateTime('2022-05-10 09:00:00');
date1.addSeconds(1000); // 2022-05-10 09:16:40

// Subtract seconds
date1 = new GlideDateTime('2022-05-10 09:00:00');
date1.addSeconds(-1000); // 2022-05-10 08:43:20
```

## Compare datetime

### Simple comparison

```javascript
var date1 = new GlideDateTime('2022-05-10 09:00:00');
var date2 = new GlideDateTime('2022-05-12 12:00:00');

if (date1 > date2) {
  gs.info('date 1 is newer than date 2');
} else {
  gs.info('date 1 is older than date 2');
}

// should print: date 1 is older than date 2
```

### After/before

```javascript
var date1 = new GlideDateTime('2022-05-10 09:00:00');
var date2 = new GlideDateTime('2022-05-12 12:00:00');

if (date1.after(date2)) {
  gs.info('date 1 is newer than date 2');
}

if (date1.before(date2)) {
  gs.info('date 1 is older than date 2');
}

// should print: date 1 is older than date 2
```

### Real world example

Let's say we want to log all incidents that hasnt been updated in the last 7 days.

```javascript
var now = new GlideDateTime();
var incident = new GlideRecord('incident');
incident.addActiveQuery();
incident.query();

while (incident.next()) {
  var lastUpdatedOn = new GlideDateTime(incident.sys_updated_on);
  lastUpdatedOn.addDaysUTC(7);

  // if current datetime is after sys_updated_on + 7 days, then we know
  // that 7 days has passed
  if (now.after(lastUpdatedOn)) {
    gs.info('Incident ' + incident.number + ' has not been updated in the last 7 days');
  }
}
```
