---
Author: carter shanklin
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6581
Published Date: 2016-10-17t14
Archived Date: 2016-12-22t03
---

# get-localgroupmembers - 

## Description

ru_dax_erp_read

## Comments



## Usage



## TODO



## function

`get-localgroupmembers`

## Code

`#
 #
 function Get-LocalGroupMembers {
 	param($groupname)
 
 	$pattern = "*Name=`"$groupname`""
 	$groupusers = gwmi Win32_GroupUser | Where { $_.GroupComponent -like $pattern }
 
 	foreach ($user in $groupusers) {
 		if ($user.PartComponent -match 'Name="([^"]+)') {
 			Write-Output $matches[1]
 		}
 	}
 }
 
`

