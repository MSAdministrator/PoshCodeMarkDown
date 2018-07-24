---
Author: andy arismendi
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2887
Published Date: 2012-08-03t09
Archived Date: 2015-05-08t16
---

# get-scriptdirectory - 

## Description

returns the directory that current script is running in.

## Comments



## Usage



## TODO



## function

`get-scriptdirectory`

## Code

`#
 #
 function Get-ScriptDirectory {   
 	$invocation = (Get-Variable MyInvocation -Scope 1).Value
 	$script = [IO.FileInfo] $invocation.MyCommand.Path
 	if ([IO.File]::Exists($script)) {
     	Return (Split-Path $script.Fullname)
 	} else {
 		return $null
 	}
 }
`

