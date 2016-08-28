---
layout: post
title:  "Readme snaptime python package"
date:   2016-07-16 17:40:30 +0200
categories: python
---

### snaptime

The snaptime package is about simply manipulating timestamps.

It is inspired by splunks relative time modifiers, see docs [here][splunk-docs].

### Motivating example:
Say Harry wants to send a letter and there is mailbox right to his door, which is cleared every day at 16:00h. The post delivers letters the next day at 11:00h. So given a datetime `t` - at which Harry throws the letter into the mailbox - when does the letter get delivered?

One straightforward solution would be:

1. check whether `t` is before or after 16:00h
2. truncate hours, minutes and seconds and
3. add 1 or 2 days (according step 1) plus 11h

This would take some lines of code, but with snaptime the solution would be a one-liner:

```python
from snaptime import snap
snap(t, "+8h@d+1d+11h")
```

In plain words this

1. adds 8 hours to `t`
2. snaps to the beginning of the day
3. adds one day and 11h

### Two types of transformations

There are two types of transformations that can be chained freely in the snap function.

#### Snap Transformation

A snap transformation is written as `@<timeunit>`, which truncates to the given time unit. I.e. `@h` applied to a timestamp sets minutes, seconds and microseconds to zero. Snapping to seconds, minutes, days, months, years work the same way.
But it is also possible to snap to weeks (`@w`), and specific week days (`@w0`, `@w1`, ..., `@w6` for Sunday, Monday, ..., Saturday)

#### Delta Transformation

A delta transformation (the name is kind of an overstatement..) works analogously to the `datetime.timedelta` object from the standard library. A signed number ([+-]\d+) and a time unit indicate the change on the given timestamp.

### Time Units

There are several strings for each time unit that indicate that same time unit. Here is a table:

| Unit | Equal notations |
|:---:|:---|
|seconds| s, sec, secs, second, seconds|
|minutes| m, min, minute, minutes|
|hours| h, hr, hrs, hour, hours|
|days| d, day, days|
|weeks| w, week, weeks|
|months| mon, month, months|
|years| y, yr, yrs, year, years|

So for a given datetime `t` this assertion holds true

```python
assert snap(t, "-3hours@day+2weeks@month") == snap(t, "-3hrs@d+2w@mon")
```

### Examples

```python
>>> from datetime import datetime
>>> from snaptime import snap

>>> dt = datetime(2016, 7, 30, 15, 23, 59)
>>> snap(dt, "@d")
datetime(2016, 7, 30, 0, 0)

>>> snap(dt, "-3d@d")
datetime(2016, 7, 27, 0, 0)
```

### Development

TODOS:

* consolidate error messages, collect all errors vs. fail fast
* refactor a bit
* motivating example: What if Harry inserts the letter at 16:00h into the mailbox? Change example?!
* add an epsilon to change the snap behaviour?!

[splunk-docs]: http://docs.splunk.com/Documentation/Splunk/latest/SearchReference/SearchTimeModifiers#How_to_specify_relative_time_modifiers
