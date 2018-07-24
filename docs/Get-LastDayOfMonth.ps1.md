---
Author: martijn jonker
Publisher: 
Copyright: 
Email: 
Version: 1.0
Encoding: ascii
License: cc0
PoshCode ID: 6497
Published Date: 2016-08-31t08
Archived Date: 2016-09-04t23
---

# get-lastdayofmonth - 

## Description

powershell function that returns one of three possible datetime objects

## Comments



## Usage



## TODO



## function

`get-lastdayofmonth`

## Code

`#
 #
 <#
 .Synopsis
 Function to find the last day, last weekday, or last named day of a month,
 based on an input date.
 
 .Description
 Function to find the last day, last weekday, or last named day of a month,
 based on an input date.
 
 .Parameter Date
 Mandatory [DateTime] parameter
 e.g. 01-01-2016
 This parameter is a member of all used parametersets
 
 .Parameter LastDay
 Mandatory [Switch] parameter
 This parameter is a member of the parameterset 'LastDay'
 
 .Parameter LastWeekDay
 Mandatory [Switch] parameter
 This parameter is a member of the parameterset 'LastWeekDay'
 
 .Parameter LastNamedDay
 Mandatory [String] parameter with validation for the days of the week
 This parameter is a member of the parameterset 'LastNamedDay'
 
 .Outputs
 A single [DateTime] object, representing the last day of the month, last weekday of the month,
 or last named day of the month.
 
 .Notes
 Author: Martijn Jonker (scriptyharry@gmail.com)
 Version: 1.0
 
 VERSION HISTORY:
 0.1: Initial creation, reusing bits and pieces found around the Internet.
 0.2: Testing functionality.
 0.3: Added LastWeekDay functionality.
 0.4: Testing functionality.
 0.5: Added LastNamedDay functionality.
 0.6: Testing functionality.
 0.7: Minor adjustment to LastNamedDay functionality.
 1.0: Satisfied with functionality, release to production
 #>
 Function Get-LastDayOfMonth {
     Param (
         [Parameter(ParameterSetName='LastDay',Mandatory = $True)]
         [Parameter(ParameterSetName='LastWeekDay',Mandatory = $True)]
         [Parameter(ParameterSetName='LastNamedDay',Mandatory = $True)]
         [DateTime]$Date,
         [Parameter(ParameterSetName='LastDay',Mandatory = $True)]
         [Switch]$LastDay,
         [Parameter(ParameterSetName='LastWeekDay',Mandatory = $True)]
         [Switch]$LastWeekDay,
         [Parameter(ParameterSetName='LastNamedDay',Mandatory = $True)]
         [ValidateSet('Monday','Tuesday','Wednesday','Thursday','Friday','Saturday','Sunday')]
         [String]$LastNamedDay
     )
     #$Today = [DateTime]::Today
     $DaysInMonth = [DateTime]::DaysInMonth($Date.Year, $Date.Month)
     $LastDayOfMonth = Get-Date -Year $Date.Year -Month $Date.Month -Day $DaysInMonth -Hour 0 -Minute 0 -Second 0
     If ($LastDay) {
         Return $LastDayOfMonth
     }
     If ($LastWeekDay) {
         $Weekdays = 1..5
         Switch ([Int]$LastDayOfMonth.DayOfWeek) {
             {$Weekdays -contains [Int]$LastDayOfMonth.DayOfWeek} {
                 $LastWeekDayOfMonth = $LastDayOfMonth
             }
             {[Int]$LastDayOfMonth.DayOfWeek -eq 6} {
                 $LastWeekDayOfMonth = $LastDayOfMonth.AddDays(-1)
             }
             {[Int]$LastDayOfMonth.DayOfWeek -eq 0} {
                 $LastWeekDayOfMonth = $LastDayOfMonth.AddDays(-2)
             }
         }
         Return $LastWeekDayOfMonth
     }
     If ($LastNamedDay){
         $NamedDay = Invoke-Expression $(-join ('[DayOfWeek]::',$LastNamedDay))
         $Diff = ([int]$NamedDay) - ([int]$LastDayOfMonth.DayOfWeek)
         If ($Diff -gt 0) {
             Return $LastDayOfMonth.AddDays(- (7-$Diff))
         }
         Else {
             Return $LastDayOfMonth.AddDays($Diff)
         } 
     }
 }
`

