---
Author: skourlatov
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5271
Published Date: 2015-07-01t09
Archived Date: 2015-01-31t20
---

# is-admin - 

## Description

verify user is administrator

## Comments



## Usage



## TODO



## function

`is-admin`

## Code

`#
 #
 Function Is-Admin
 {
 	$principal = [Security.Principal.WindowsPrincipal][Security.Principal.WindowsIdentity]::GetCurrent()
 	$role = [Security.Principal.WindowsBuiltInRole]::Administrator
 	return $principal.IsInRole($role)
 }
`

