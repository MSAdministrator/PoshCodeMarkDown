---
Author: dragonmc77
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5657
Published Date: 2015-12-30t18
Archived Date: 2015-11-24t08
---

# get-uncpath - 

## Description

simple function that returns the unc path (administrative share) of a local path.

## Comments



## Usage



## TODO



## function

`get-uncpath`

## Code

`#
 #
 function Get-UNCPath {param(	[string]$HostName,
 				[string]$LocalPath)
 	$NewPath = $LocalPath -replace(":","$")
 	if ($NewPath.EndsWith("\")) {
 		$NewPath = [Text.RegularExpressions.Regex]::Replace($NewPath, "\\$", "")
 	}
 	"\\$HostName\$NewPath"
 }
`

