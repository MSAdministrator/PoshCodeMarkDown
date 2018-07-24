---
Author: wayne johnson
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 4741
Published Date: 2016-12-26t22
Archived Date: 2016-10-18t11
---

# delete old shadow copies - 

## Description

i use this script to delete shadow copies older than 30 days from our file and print servers.  i have it installed on server 2012 core servers.

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 
 Get-WmiObject Win32_Shadowcopy | ForEach-Object {
 
 $WmiSnapShotDate = $_.InstallDate
 $strShadowID = $_.ID
 $dtmSnapShotDate = [management.managementDateTimeConverter]::ToDateTime($WmiSnapShotDate) 
 $strClientAccessible = $_.ClientAccessible
 $dtmCurDate = Get-Date
 
 $dtmTimeSpan = New-TimeSpan $dtmSnapShotDate $dtmCurDate 
 $intNumberDays = $dtmTimeSpan.Days
 
 If ($intNumberDays -ge 31 -and $strClientAccessible -eq "True") {
   
   $_.Delete()
 }
 
 }
`

