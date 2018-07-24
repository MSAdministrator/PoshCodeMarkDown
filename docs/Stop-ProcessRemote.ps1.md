---
Author: brian wahoff
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6659
Published Date: 2016-12-23t22
Archived Date: 2016-12-31t02
---

# stop-processremote - 

## Description

in an interesting design choice, get-process lets you work with processes on remote machines, but stop-process does not. this cmdlet uses wmi to stop a process on a remote machine.

## Comments



## Usage



## TODO



## function

`stop-processremote`

## Code

`#
 #
 Function Stop-ProcessRemote()
 {
 <#
 .SYNOPSIS
 	Stops a process on a remote computer
 .DESCRIPTION
 	Uses WMI to connect to a remote computer and terminate a process.
 	Assumes the user has administrative priviledges on the remote 
 	computer.
 .NOTES
 	Author      : Brian Wahoff
 	Requires    : Powershell V2
 .PARAMETER ComputerName
 	The remote computer to which you want to connect
 .PARAMETER Id
 	The PID of the process to stop (See Get-Process)
 .PARAMETER ProcessName
 	The name of the process to stop. Will stop all processes with the same name
 #>
 	param(
 		[Parameter(Position=0, Mandatory=$TRUE)]
 		[string]$ComputerName, 
 		
 		[Parameter(ParameterSetName="p1",Position=1,ValueFromPipeline=$TRUE)]
 		[int]$Id,
 		
 		[Parameter(ParameterSetName="p2",Position=1, ValueFromPipeline=$TRUE)]
 		[string]$ProcessName)
 			
 	if ($Id) {
 		$query = "select * from Win32_Process Where ProcessID = {0}" -f $Id
 	} else {
 		if ($ProcessName) {
 			$query = "select * from Win32_Process Where Name = '{0}'" -f $ProcessName
 		} else {
 			throw 'Either $Id or $ProcessName is required'
 		}
 	}
 	
 	$process = Get-WMIObject -computer $ComputerName -query $query
 	if ($process) {
 		if ($process.count -gt 1) {
 			foreach ($p in $process) {
 				Stop-WMIProcess($p)
 			}
 		} else {
 			Stop-WMIProcess($process)
 		}
 	} else {
 		if ($ProcessName)
 		{
 			"Process '{0}' was not running on \\{1}" -f $ProcessName, $ComputerName
 		} else {
 			"Process '{0}' was not running on \\{1}" -f $Id, $ComputerName
 		}
 	}
 }
 
 Function Stop-WMIProcess($WmiProcess) {
 <#
 .SYNOPSIS
 	Stop a WmiProcess
 .DESCRIPTION
 	Wrapper function around WmiProcess.Terminate. Displays message 
 	based on all documented return values. Not intended to be called
 	directly.
 .NOTES
 	Author		: Brian Wahoff
 	Requires	: Powershell V2
 .PARAMETER WmiProcess
 	The WMI Process object to terminate
 #>
 	$ret = $WmiProcess.Terminate()
 	
 	switch ($ret.ReturnValue)
 	{
 		0 {
 			"Process {0}:{1} terminated" -f $WmiProcess.Name, $WmiProcess.ProcessID
 		}
 		2 {
 			"Access was denied terminating {0}" -f $WmiProcess.Name
 		}
 		3 {
 			"Insufficient Privilege terminating {0}" -f $WmiProcess.Name
 		}
 		8 {
 			"Unknown failure terminating {0}" -f $WmiProcess.Name
 		}
 		9
 		{
 			"Path Not Found"
 		}
 		21
 		{
 			"WMI Parameter Invalid"
 		}
 	}
 }
`

