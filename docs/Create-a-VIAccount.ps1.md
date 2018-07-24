---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 1517
Published Date: 
Archived Date: 2009-12-12t18
---

# create a viaccount - 

## Description

create a viaccount object suitable for use with new-vipermission, get-vipermission, etc. from powercli.

## Comments



## Usage



## TODO



## function

`new-viaccount`

## Code

`#
 #
 function New-VIAccount($principal) {
 	$flags = `
 		[System.Reflection.BindingFlags]::NonPublic    -bor
 		[System.Reflection.BindingFlags]::Public       -bor
 		[System.Reflection.BindingFlags]::DeclaredOnly -bor
 		[System.Reflection.BindingFlags]::Instance
 	$method = $defaultviserver.GetType().GetMethods($flags) |
 		where { $_.Name -eq "VMware.VimAutomation.Types.VIObjectCore.get_Client" }
 	$client = $method.Invoke($global:DefaultVIServer, $null)
 	Write-Output `
 		(New-Object VMware.VimAutomation.Client20.PermissionManagement.VCUserAccountImpl `
 			-ArgumentList $principal, "", $client)
 }
`

