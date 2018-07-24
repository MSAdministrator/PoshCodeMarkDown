---
Author: tony sathre
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 4201
Published Date: 2013-06-13t02
Archived Date: 2013-11-27t06
---

# get-uptime - 

## Description

get the system uptime of the localhost or a remote host.

## Comments



## Usage



## TODO



## class

``

## Code

`#
 #
 Param (
     [string]$computerName = $env:computerName
 )
 
 $os = Get-WmiObject -ComputerName $computerName -Class win32_operatingsystem
 $boottime = [management.managementDateTimeConverter]::ToDateTime($os.LastBootUpTime)
 $now = [management.managementDateTimeConverter]::ToDateTime($os.localdatetime)
 $uptime = New-TimeSpan -Start $boottime -End $now
 if ($computername -ne $os.csname) { $altname = " ($($os.csname))" }
 Write-Host "Uptime on ${computerName}${altname}`:" $uptime.Days 'Days,' $uptime.Hours 'Hours,' $uptime.Minutes 'Minutes,' $uptime.Seconds 'Seconds'
`

