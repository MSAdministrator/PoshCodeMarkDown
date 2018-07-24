---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 2.0
Encoding: ascii
License: gpl, ms-rl, ms-pl, or bsd
PoshCode ID: 966
Published Date: 2009-03-19t22
Archived Date: 2016-04-22t18
---

# isodates.psm1 - 

## Description

a handful of functions for returning is0-8601 formatted dates and parsing them. all functions work (even in the pipeline) in both powershell 1 and 2.

## Comments



## Usage



## TODO



## function

`get-isodate`

## Code

`#
 #
 ########################################################################
 function Get-ISODate {
 #.Synopsis
 #.Description
 #.Parameter Date
 Param([DateTime]$date=$(Get-Date))
 Process {
   if($_){$date=$_}
   $startYear  = Get-Date -Date $date -Month 1 -Day 1 
   $endYear    = Get-Date -Date $date -Month 12 -Day 31 
   $dayOfWeek  = @(7,1,2,3,4,5,6)[$date.DayOfWeek]
   $lastofWeek = @(7,1,2,3,4,5,6)[$endYear.DayOfWeek]
   $adjust     = @(6,7,8,9,10,4,5)[$startYear.DayOfWeek]
   
   switch([Math]::Floor((($date.Subtract($startYear).Days + $adjust)/7)))
   {
     0 {
       Write-Output @(
          @(Get-ISODate $startYear.AddDays(-1))[0,1] + @(,$dayOfWeek))
     }
     53 {
       if ($lastofWeek -lt 4) {
         Write-Output @(($date.Year + 1), 1, $dayOfWeek)
       } else {
         Write-Output @($date.Year, $_, $dayOfWeek)
       }
     }
     default { 
       Write-Output @($date.Year, $_, $dayOfWeek)
     }
   }
 }
 }
 
 function Get-ISOWeekOfYear {
 #.Synopsis
 #.Description
 #.Parameter Date
 Param([DateTime]$date=$(Get-Date))
 Process {
   if($_){$date=$_}
   @(Get-ISODate $date)[1]
 }
 }
 
 function Get-ISOYear {
 #.Synopsis
 #.Description
 #.Parameter Date
 Param([DateTime]$date=$(Get-Date))
 Process {
   if($_){$date=$_}
   @(Get-ISODate $date)[0]
 }
 }
 
 function Get-ISODayOfWeek {
 #.Synopsis
 #.Description
 #.Parameter Date
 Param([DateTime]$date=$(Get-Date))
 Process {
   if($_){$date=$_}
   @(7,1,2,3,4,5,6)[$date.DayOfWeek]
 }
 }
 
 function Get-ISODateString {
 #.Synopsis
 #.Description
 #.Parameter Date
 Param([DateTime]$date=$(Get-Date))
 Process {
   if($_){$date=$_}
   "{0}-W{1:00}-{2}" -f @(Get-ISODate $date)
 }
 }
 
 function ConvertFrom-ISODateString {
 #.Synopsis
 #.Description
 #.Parameter Date
 Param([string]$date)
 Process {
   if($_){$date=$_}
   if($date -notmatch "\d{4}-W\d{2}-\d") {
     Write-Error "The string is not an ISO 8601 formatted date string"
   }
   $ofs = ""
   $year = [int]"$($date[0..3])"
   $week = [int]"$($date[6..7])"
   $day  = [int]"$($date[9])"
 
   $firstOfYear = Get-Date -year $year -day 1  -month 1
   $days = 7*$week - ((Get-ISODayOfWeek $firstOfYear) - $day)
  
   $result = $firstOfYear.AddDays( $days )
   if(($result.Year -ge $year) -and ((Get-ISODayOfWeek $firstOfYear) -le 4) ) {
    return $firstOfYear.AddDays( ($days - 7) )
   } else {
    return $result
   }
 }
 }
`

