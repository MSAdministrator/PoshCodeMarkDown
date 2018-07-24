---
Author: steve jarvi
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 4502
Published Date: 2014-10-01t21
Archived Date: 2017-04-30t10
---

# get-wmievent - 

## Description

this function handles wmi events based on a query against remote systems.  no polling, no looping or do-whileing.  includes sample queries.

## Comments



## Usage



## TODO



## script

`get-wmievent`

## Code

`#
 #
         WMI Events Handler
 	   .DESCRIPTION
 		Handles WMI Events Based on Queries
        .PARAMETER
 		
        .INPUTS
 	   $computername, $Query
 			$Query = "select * from __instanceCreationEvent within 10 where targetInstance isa 'win32_Process' and targetInstance.name = 'Firefox.exe'"
 			$Query = "select * from __instanceCreationEvent within 10 where targetInstance isa 'win32_Process'"
 			$Query = "select * from __instanceDeletionEvent within 10 where targetInstance isa 'win32_Process' and targetInstance.ProcessID = '1143'"
 	   .OUTPUTS
 	   .NOTES
         Name: Get-WMIEvent
         Author: Steve Jarvi
         DateCreated: 10 Sept 2013
 	   .EXAMPLE
     #>
 Function Get-WMIEvent {
 param (
 [string]$computername
 [string]$query
 )
 
 $ManagementScope = New-Object management.ManagementScope("\\$computername\root\cimv2")
 $Eventwatcher = New-Object management.managementEventWatcher($ManagementScope, $Query)
 
 $Event = $Eventwatcher.waitForNextEvent()
 $Event.targetinstance
 $Eventwatcher.start()
 }
`

