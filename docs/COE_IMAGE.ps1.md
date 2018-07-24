---
Author: gerald klassen
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2507
Published Date: 2011-02-15t13
Archived Date: 2016-05-20t10
---

# coe_image - 

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

