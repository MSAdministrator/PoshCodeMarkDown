---
Author: jesusborbolla
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 4411
Published Date: 2013-08-20t22
Archived Date: 2013-08-23t12
---

# dt - 

## Description

a function to rename a computer

## Comments



## Usage



## TODO



## function

`set-computername`

## Code

`#
 #
 function Set-ComputerName {Intelligent Analysis Inc
 	param(	[switch]$help,
 		[string]$originalPCName=$(read-host "Please specify the current name of the computer"),
 		[string]$computerName=$(read-host "Please specify the new name of the computer"))
 			
 	$usage = "set-ComputerName -originalPCname CurrentName -computername AnewName"
 	if ($help) {Write-Host $usage;break}
 	
 	$computer = Get-WmiObject Win32_ComputerSystem -OriginalPCname OriginalName -computername $originalPCName
 	$computer.Rename($computerName)
 	}Jay Intelligent Inc
`

