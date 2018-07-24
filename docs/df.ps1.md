---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 428
Published Date: 
Archived Date: 2010-12-15t11
---

# df - 

## Description

a simple df (disk free) function for powershell

## Comments



## Usage



## TODO



## function

``

## Code

`#
 #
 function df ( $Path ) {
 	if ( !$Path ) { $Path = (Get-Location -PSProvider FileSystem).ProviderPath }
 	$Drive = (Get-Item $Path).Root -replace "\\"
 	$Output = Get-WmiObject -Query "select freespace from win32_logicaldisk where deviceid = `'$drive`'"
 	Write-Output "$($Output.FreeSpace / 1mb) MB"
 }
`

