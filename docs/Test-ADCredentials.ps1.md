---
Author: mike pfeiffer
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6267
Published Date: 2016-03-24t12
Archived Date: 2016-10-24t17
---

# test-adcredentials - 

## Description

validates a username and password against active directory. requires .net 3.5 and powershell v2.

## Comments



## Usage



## TODO



## function

`test-adcredentials`

## Code

`#
 #
 Function Test-ADCredentials {
 	Param($username, $password, $domain)
 	Add-Type -AssemblyName System.DirectoryServices.AccountManagement
 	$ct = [System.DirectoryServices.AccountManagement.ContextType]::Domain
 	$pc = New-Object System.DirectoryServices.AccountManagement.PrincipalContext($ct, $domain)
 	New-Object PSObject -Property @{
 		UserName = $username;
 		IsValid = $pc.ValidateCredentials($username, $password).ToString()
 	}
 }
`

