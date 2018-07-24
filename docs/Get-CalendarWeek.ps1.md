---
Author: holger adam
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 1392
Published Date: 2010-10-13t07
Archived Date: 2016-04-22t18
---

# get-calendarweek - 

## Description

this function calculates the calendar week to a given date. it either takes a given date or retrieves the current date.

## Comments



## Usage



## TODO



## function

`get-calendarweek`

## Code

`#
 #
 
 function Get-CalendarWeek {
 	param(
 		$Date = (Get-Date)
 	)
 	
 	$Culture = [System.Globalization.CultureInfo]::CurrentCulture
 	
 	$Culture.Calendar.GetWeekOfYear($Date, $Culture.DateTimeFormat.CalendarWeekRule, $Culture.DateTimeFormat.FirstDayOfWeek)
 }
`

