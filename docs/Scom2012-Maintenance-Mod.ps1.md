---
Author: ninjatechie
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 4697
Published Date: 2013-12-13t13
Archived Date: 2013-12-16t12
---

# scom2012 maintenance mod - 

## Description

script to put several machines into maintenance mode on your opsmgr2012 instance.

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 #########################################
 #
 #########################################
  
 $OMServer = "operationsmanagerserver.contoso.local"
 $domain = "contoso.local"
 $minutes = 240
 $computers = '.\Computers.txt'
 $class = "Microsoft.Windows.Computer"
 $comment = 'Scheduled Maintenance'
  
 Import-Module OperationsManager
 New-SCOMManagementGroupConnection $OMServer
  
 $class = Get-SCOMClass -Name $class
 $startTime = [System.DateTime]::Now.ToUniversalTime()
 $endTime = [System.DateTime]::Now.AddMinutes($minutes).ToUniversalTime()
  
 $ComputerNames = Get-Content $computers
  
 foreach ($name in $ComputerNames) {
  $instance = Get-SCOMClassInstance -Class $class | ? { $_.Name -eq ($name + "." + $domain) }
  "Putting " + $name + " into maintenance mode..."
  $instance.ScheduleMaintenanceMode($startTime, $endTime, "PlannedApplicationMaintenance", $comment , "Recursive")
  Start-Sleep -s 2
 }
`

