---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2397
Published Date: 
Archived Date: 2010-12-10t22
---

# get-cpl - 

## Description

a function to retrieve available control panel applets along with a description.

## Comments



## Usage



## TODO



## function

`get-cpl`

## Code

`#
 #
 function Get-Cpl {
 dir $env:windir\system32 | Where-Object {$_.Extension -eq ".cpl"} | Select-Object Name,@{Name="Description";Expression={$_.VersionInfo.FileDescription}} | Sort-Object Description | Format-Table -AutoSize
 }
`

