---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 1.2
Encoding: ascii
License: cc0
PoshCode ID: 3059
Published Date: 2011-11-20t21
Archived Date: 2011-11-29t18
---

# new-event - 

## Description

a powershell client for naturalinputs.com to parse natural language dates into (optionally recurring) calendar event objects.

## Comments



## Usage



## TODO



## function

`new-event`

## Code

`#
 #
 function New-Event {
 <#
 .Synopsis
    Generates a new event from a plain English string
 .Description
    Takes a plain English date-time string like "Scrum every day at 9am" or "Lunch every other Tuesday at noon" and turns it into an event object (see notes).
 .Example
    New-Event Lunch every other Friday at 11:45
 .Example
    New-Event Lunch at noon on Christmas
 .Notes
    v 1.2 Supports resolving common calendar holidays
          But NaturalInputs' API still can't do relative dates, so:
          You can now schedule "Lunch at noon on Easter" but not "Lunch at noon the Sunday before Memorial Day" or "Dinner at 6 PM the day after Christmas"
 #>
    param( 
       [Parameter(ValueFromRemainingArguments=$true, ValueFromPipeline=$true, Position=0)]
       [string]$query
    ,
       [DateTime]$currentTime = $([DateTime]::Now)
    ,
       [ValidateSet("Canada", "GreatBritain", "IrelandNorthern", "IrelandRepublicOf", "Scotland", "UnitedStates")]
       [String]$Country = "UnitedStates"
    )
    begin {
       $wc = New-Object System.Net.WebClient
       $time = $currentTime.ToString("yyyyMMddThhmmss")
       
       if(!$global:Holidays) {
          $service = New-WebServiceProxy http://www.holidaywebservice.com/HolidayService_v2/HolidayService2.asmx
          
          $global:Holidays  = $service.GetHolidaysForYear( $Country, $currentTime.Year )
          $global:Holidays += $service.GetHolidaysForYear( $Country, $currentTime.Year + 1 )
       }
       $HolidayKeys = @{}
       foreach($holiday in $global:Holidays) {
          $key = $holiday.Descriptor -replace " Day" -replace "'s" -replace " Birthday"
          $key = $key.Trim()
          $HolidayKeys[$key] += @($holiday)
       }
    }
    process {
       
       foreach($key in $HolidayKeys.Keys) {
          $keyPattern = [System.Text.RegularExpressions.Regex]::Escape($key)
          if($query -match $keyPattern ) {
             Write-Verbose $keyPattern
             if($query -match "next\s+$keyPattern") {
                $currentTime = $currentTime.AddYears(1)
             }
             if($query -match "$keyPattern.*observed") {
                $holiday = $HolidayKeys[$key] | Where-Object { $_.DateType -match "observed" -and ($_.Date.Year -eq $currentTime.Year) } | Select -First 1
             } else {
                $holiday = $HolidayKeys[$key] | Where-Object { $_.DateType -match "actual" -and ($_.Date.Year -eq $currentTime.Year) } | Select -First 1
             }
             $query = $query -replace "(next\s+)?$keyPattern(.*observed)?", $Holiday.Date.ToString("d",$Culture)
             break
          }
       }
 
       Write-Verbose "Query: $query"
       $query = [System.Web.HttpUtility]::UrlEncode( $query )
       [xml]$event = $wc.DownloadString( "http://naturalinputs.com/query?t=$time&q=$query" )
       Write-Verbose "Response: $($event.OuterXml)"
       
       foreach($r in $event.naturalinputsresponse) {
          foreach($o in $r.occurrence) {
             if($o.start_date) {
                if($o.start_time) {
                   $startdate = Get-Date ([DateTime]::ParseExact( ("{0}T{1}" -f $o.start_date, $o.start_time), "yyyyMMddTHH:mm:ss", $null)) -DisplayHint "Date"
                   $starttime = Get-Date ([DateTime]::ParseExact( ("{0}T{1}" -f $o.start_date, $o.start_time), "yyyyMMddTHH:mm:ss", $null)) -DisplayHint "Time"
                } else {
                   $startdate = Get-Date ([DateTime]::ParseExact( $o.start_date, "yyyyMMdd", $null)) -DisplayHint "Date"
                   $starttime = $null
                }
             } elseif ($o.start_time) {
                $startdate = $null
                $starttime = Get-Date $o.start_time -DisplayHint "Time"
             }
             
             if($o.end_date) {
                if($o.end_time) {
                   $enddate = Get-Date ([DateTime]::ParseExact( ("{0}T{1}" -f $o.end_date, $o.end_time), "yyyyMMddTHH:mm:ss", $null)) -DisplayHint "Date"
                   $endtime = Get-Date ([DateTime]::ParseExact( ("{0}T{1}" -f $o.end_date, $o.end_time), "yyyyMMddTHH:mm:ss", $null)) -DisplayHint "Time"
                } else {
                   $enddate = Get-Date ([DateTime]::ParseExact( $o.end_date, "yyyyMMdd", $null)) -DisplayHint "Date"
                   $endtime = $null
                }
             } elseif ($o.end_time) {
                if($o.start_date) {
                   $enddate = $null
                   $endtime = Get-Date ([DateTime]::ParseExact( ("{0}T{1}" -f $o.start_date, $o.end_time), "yyyyMMddTHH:mm:ss", $null)) -DisplayHint "Time"
                } else {
                   $enddate = $null
                   $endtime = Get-Date $o.end_time -DisplayHint "Time"
                }
             }
 
             New-Object PSObject -Property @{
                Name=$r.message
                Type=$o.type
                DayOfWeek=$o.day_of_week
                WeekOfMonth=$o.week_of_month
                DateOfMonth=$o.date_of_month
                Interval=$o.interval
                StartDate=$startdate
                StartTime=$starttime
                EndDate=$enddate
                EndTime=$endtime
             } | Add-Member ScriptProperty TimeUntil { $this.StartDate - (Get-Date) } -Passthru
          }
       }
 
    }
 }
`

