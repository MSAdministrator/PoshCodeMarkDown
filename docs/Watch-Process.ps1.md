---
Author: ravikanth
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2560
Published Date: 2011-03-15t06
Archived Date: 2016-08-11t17
---

# watch-process - 

## Description

creates an event handler for monitoring either process creation or deletion. this requires to be run as administrator.

## Comments



## Usage



## TODO



## function

`watch-process`

## Code

`#
 #
 Function Watch-Process {
 	<#
 	.DESCRIPTION
 		Creates an event handler for monitoring either process creation or deletion. This requires to be run as administrator.
 	.SYNOPSIS
 		Watches for process creation or deletion.
 	.PARAMETER computerName
 		Name of the remote computer. Make sure you have privileges to access remote WMI namespaces. 
         The default value is local computer.
 	.PARAMETER Name
 		Name of the process to monitor.
 	.PARAMETER Id
 		Processs ID of the process to monitor.
     .PARAMETER Creation
         Switch Parameter. Use this to start process creation monitor.
     .PARAMETER Deletion
         Switch Parameter. Use this to start process deletion monitor.
     .PARAMETER Timeout
         By default there is no timeout. The process monitor will wait forever. You can specify the maximum timeout period in seconds.
 	.OUTPUTS
 		Returns a process object in case of process creation
         and returns process exit status in case of process deletion
 	.EXAMPLE
 		Watch-Process -computerName TestServer01 -Name "Notepad.exe" -Creation
 		
 		Description
 		-----------
 		The above example demonstrates to how to start a process creation monitor for a remote process
 	.EXAMPLE
 		Watch-Process -computerName TestServer01 -Name "notepad.exe" -Deletion
         Watch-Process -computerName TestServer01 -Id 3123 -Deletion
 		
 		Description
 		-----------
 		The above creates process deletion monitor for notepad.exe on computer TestServer01 and also creates a process deletion monitor for process ID 3123 on the remote computer.
 	.LINK
 		Online version: http://www.ravichaganti.com/blog
 	#>
 	[CmdletBinding()]
 	param (
 		[Parameter(ParameterSetName="pCreation",Mandatory=$false)]
         [Parameter(ParameterSetName="pDeletion",Mandatory=$false)]
     	[String]$computerName=".",
 		
 		[Parameter(ParameterSetName="pCreation",Mandatory=$true)]
         [Parameter(ParameterSetName="pDeletion",Mandatory=$false)]
     	[String]$name,
 		
 		[Parameter(ParameterSetName="pDeletion",Mandatory=$false)]
     	[int]$Id,
         
         [Parameter(ParameterSetName="pCreation",Mandatory=$false)]
     	[Switch]$creation,
         
         [Parameter(ParameterSetName="pDeletion",Mandatory=$false)]
     	[Switch]$deletion,
         
         [Parameter(ParameterSetName="pDeletion",Mandatory=$false)]
         [Parameter(ParameterSetName="pCreation",Mandatory=$false)]
     	[int]$timeout=-1
 	)
         
     if ($deletion) {
         if (($PSBoundParameters.Keys -contains "Name") -and ($PSBoundParameters.Keys -Contains "Id")) {
             Write-Error "Both Name and Id parameters are specified. Specify any of these parameters."
             return
         } elseif ($name) {
             $query = "SELECT * FROM Win32_ProcessStopTrace WHERE ProcessName='$($name)'"
             Write-Verbose $query
         } elseif ($id) {
             $query = "SELECT * FROM Win32_ProcessStopTrace WHERE ProcessID='$($Id)'"
             Write-Verbose $query
         } else {
             Write-Error "Neither -Name nor -Id provided. You must provide one of these parameters."
             return
         }
         
     } elseif ($creation) {
         $query = "SELECT * FROM Win32_ProcessStartTrace WHERE ProcessName='$($name)'"
         Write-Verbose $query
     } else {
         Write-Error "You must specify an event to monitor. The valid parameters are -deletion or -creation"
         return
     }
     
     if ($query) {
         $srcId = [guid]::NewGuid()
         Write-Verbose "Registering a WMI event"
         Register-WmiEvent -ComputerName $computerName -Query $query  -SourceIdentifier $srcID
             
         Wait-Event -SourceIdentifier $srcID -Timeout $timeout
             
         Write-Verbose "Unregistering a WMI event"
         Unregister-Event -SourceIdentifier $srcID
     }
 }
`

