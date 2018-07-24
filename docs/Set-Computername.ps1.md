---
Author: steven
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6036
Published Date: 2016-10-04t10
Archived Date: 2016-05-17t07
---

# set-computername - 

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
 function Set-ComputerName {
 	param(	[switch]$help,
 		[string]$originalPCName=$(read-host "Please specify the current name of the computer"),
 		[string]$computerName=$(read-host "Please specify the new name of the computer"))
 			
 	$usage = "set-ComputerName -originalPCname CurrentName -computername AnewName"
 	if ($help) {Write-Host $usage;break}
 	
 	$computer = Get-WmiObject Win32_ComputerSystem -OriginalPCname OriginalName -computername $originalPCName
 	$computer.Rename($computerName)
 	}
`

