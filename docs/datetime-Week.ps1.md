---
Author: dale thompson
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6500
Published Date: 2016-09-01t15
Archived Date: 2016-09-03t23
---

# [datetime].week - 

## Description

this bit of code adds a week scriptproperty to datetime objects. the property returns the week of the year based on the current cultural settings.

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 Update-TypeData -Force -TypeName DateTime -MemberType ScriptProperty -MemberName Week -Value {
    [System.Globalization.CultureInfo]$Culture = [System.Globalization.CultureInfo]::CurrentCulture
    return $Culture.Calendar.GetWeekOfYear($this, $Culture.DateTimeFormat.CalendarWeekRule, $Culture.DateTimeFormat.FirstDayOfWeek)
 }
`

