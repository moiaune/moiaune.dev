---
title: 'Golang: Format Date and Time'
date: 2021-08-13T08:23:11+02:00
tags: ['golang']
draft: false
---

Most programming languages use the same layout (dd-mm-yyyy) to format date and time, but Go decided to go a different route. Below is a little cheat sheet of how to format date and time in Go.

## Examples

### Parsing exisiting date

```golang
var (
    timeToParse = "2021-09-13T07:43:52.823"
    layout      = "2006-01-02T03:04:05.999"
)

toTime, _ := time.Parse(layout, timeToParse)

fmt.Printf("(%T): %s\n", toTime, toTime)

// output: (time.Time): 2021-09-13 07:43:52.823 +0000 UTC
```

### Formatting date

```golang
now := time.Now()
fmt.Println("Default:", now)
fmt.Println("Formatted:", now.Format("02-01-2006 15:04:05 -0700 MST"))

// output: Default: 2021-08-13 09:01:29.233757 +0200 CEST m=+0.000065018
// output: Formatted: 13-08-2021 09:01:29 +0200 CEST
```

## Options

| Type     | Options                       |
| -------- | ----------------------------- |
| Year     | 06 2006                       |
| Month    | 01 1 Jan January              |
| Day      | 02 2 \_2                      |
| Weekday  | Mon Monday                    |
|          |                               |
| Hours    | 03 3 15                       |
| Minutes  | 04 4                          |
| Seconds  | 05 5                          |
|          |                               |
| ms μs ns | .000 .000000 .000000000       |
| ms μs ns | .999 .999999 .999999999       |
|          |                               |
| am / pm  | PM pm                         |
| Timezone | MST                           |
| Offset   | -0700 -07 -07:00 Z0700 Z07:00 |

## Predefined layouts

| Name        | Layout                                                       |
| ----------- | ------------------------------------------------------------ |
| ANSIC       | Mon Jan \_2 15:04:05 2006                                    |
| UnixDate    | Mon Jan \_2 15:04:05 MST 2006                                |
| RubyDate    | Mon Jan 02 15:04:05 -0700 2006                               |
| RFC822      | 02 Jan 06 15:04 MST                                          |
| RFC822Z     | 02 Jan 06 15:04 -0700 // RFC822 with numeric zone            |
| RFC850      | Monday, 02-Jan-06 15:04:05 MST                               |
| RFC1123     | Mon, 02 Jan 2006 15:04:05 MST                                |
| RFC1123Z    | Mon, 02 Jan 2006 15:04:05 -0700 // RFC1123 with numeric zone |
| RFC3339     | 2006-01-02T15:04:05Z07:00                                    |
| RFC3339Nano | 2006-01-02T15:04:05.999999999Z07:00                          |
| Kitchen     | 3:04PM                                                       |
|             |                                                              |
| Stamp       | Jan \_2 15:04:05                                             |
| StampMilli  | Jan \_2 15:04:05.000                                         |
| StampMicro  | Jan \_2 15:04:05.000000                                      |
| StampNano   | Jan \_2 15:04:05.000000000                                   |
