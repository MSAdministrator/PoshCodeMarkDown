---
Author: aleksandar
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 1078
Published Date: 2010-05-04t14
Archived Date: 2017-04-08t23
---

# get-hotfix - 

## Description

the get-hotfix function gets the quick-fix engineering (qfe) updates that have been applied to the local computer or to remote computers and filter those hotfixes named “file 1”.

## Comments



## Usage



## TODO



## function

`get-hotfix`

## Code

`#
 #
 
 function Get-HotFix {
 	param (
 	[string[]]$ComputerName = ".",
 	[string]$Id = "",
 	[string]$Description = "",
 	$credential
 	)
 
 	if ($credential -is [String] ) {
 		$credential = Get-Credential $credential
 	}
 
 
 	if ($id -ne "" -and $Description -eq "") {
 		$id = $id.Replace("*","%");$filter = "hotfixid LIKE '$id'"
 	}
 	elseif ($id -eq "" -and $Description -ne "") {
 		$Description = $Description.Replace("*","%");$filter = "description LIKE '$Description'"
 	}
 	elseif ($id -ne "" -and $Description -ne "") {
 		Write-Host "WARNING: Use Id or Description parameter." -foregroundcolor yellow ;break
 	}
 	else {
 		$filter = ""
 	}
 
 	if ($credential -eq $null) {
 		$HotFixes = get-wmiobject Win32_QuickFixEngineering -computerName $ComputerName -filter $filter | where-object {$_.hotfixid -ne "file 1"}
 	} else {
 		$HotFixes = get-wmiobject Win32_QuickFixEngineering -computerName $ComputerName -filter $filter -credential $credential | where-object {$_.hotfixid -ne "file 1"}
 	}
 
 	$HotFixes
 }
 
 #
 #
 #
`

