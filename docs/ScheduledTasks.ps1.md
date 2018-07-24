---
Author: christophecremon
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2432
Published Date: 2011-01-03t10
Archived Date: 2015-04-29t18
---

# scheduledtasks - 

## Description

powershell module to manage windows scheduled tasks

## Comments



## Usage



## TODO



## module

`get-scheduledtasks`

## Code

`#
 #
 
 Function Get-ScheduledTasks
 {
 	[CmdletBinding()]
 	param (
 		[ValidateNotNullOrEmpty()]
 		[string] $TaskName,
 		[string] $HostName )
 	
 	process
 	{
 		if ( $HostName ) { $HostName = "/S $HostName" }
 		$ScheduledTasks = SCHTASKS.EXE /QUERY /FO CSV /NH $HostName
 		foreach ( $Item in $ScheduledTasks )
 		{
 			if ( $Item -ne "" )
 			{
 				$Item = $Item -replace("""|\s","")
 				$SplitItem = $Item -split(",")
 				$ScheduledTaskName = $SplitItem[0]
 				$ScheduledTaskStatus = $SplitItem[3]
 				if ( $ScheduledTaskName -ne "" )
 				{
 					if ( $ScheduledTaskStatus -eq "" )
 					{
 						$ScheduledTaskStatus = "Not Running"
 					}
 					else
 					{
 						$ScheduledTaskStatus = "Running"
 					}
 					$objScheduledTaskName = New-Object System.Object
 	    			$objScheduledTaskName | Add-Member -MemberType NoteProperty -Name TaskName -Value $ScheduledTaskName
 					$objScheduledTaskName | Add-Member -MemberType NoteProperty -Name TaskStatus -Value $ScheduledTaskStatus
 					$objScheduledTaskName | Where-Object { $_.TaskName -match $TaskName }
 				}
 			}
 		}
 	}
 }
 Function Start-ScheduledTask
 {
 	[CmdletBinding()]
 	param (
 		[ValidateNotNullOrEmpty()]
 		[Parameter(ValueFromPipelineByPropertyName=$true,Mandatory=$true)]
 		[string] $TaskName,
 		[string] $HostName )
 
 	process
 	{
 		if ( $HostName ) { $HostName = "/S $HostName" }
 		SCHTASKS.EXE /RUN /TN $TaskName $HostName
 	}
 }
 Function Stop-ScheduledTask
 {
 	[CmdletBinding()]
 	param (
 		[ValidateNotNullOrEmpty()]
 		[Parameter(ValueFromPipelineByPropertyName=$true,Mandatory=$true)]
 		[string] $TaskName,
 		[string] $HostName )
 
 	process
 	{
 		if ( $HostName ) { $HostName = "/S $HostName" }
 		SCHTASKS.EXE /END /TN $TaskName $HostName
 	}
 }
 Export-ModuleMember Get-ScheduledTasks, Start-ScheduledTask, Stop-ScheduledTask
`

