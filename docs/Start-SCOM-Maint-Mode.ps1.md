---
Author: austin greca
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6060
Published Date: 2016-10-21t18
Archived Date: 2016-05-17t10
---

# start scom maint mode - 

## Description

turns on maintenance mode for a specific computer monitored by scom (system center operations manager).  ensure that the operationsmanager module is available on the computer from which this script is executed.

## Comments



## Usage



## TODO



## module

``

## Code

`#
 #
 Import-Module OperationsManager
 
 $computer = "mycomputer.mydomain.com"
 
 $time = (Get-date).AddMinutes(15)
 
 $instance = Get-SCOMClassInstance -Name $computer -ComputerName myscommgmtserver
 
 Start-SCOMMaintenanceMode -Instance $instance -EndTime $time -Comment "Applying updates" -Reason PlannedOther
`

