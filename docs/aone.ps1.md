---
Author: andy schneider
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 1924
Published Date: 
Archived Date: 2010-06-21t02
---

# aone - 

## Description

a function to join a domain

## Comments



## Usage



## TODO



## function

`set-domain`

## Code

`#
 #
 function Set-Domain {
 	param(	[switch]$help,
 			[string]$domain=$(read-host "Please specify the domain to join"),
 			[System.Management.Automation.PSCredential]$credential = $(Get-Crdential) 
 			)
 			
 	$usage = "`$cred = get-credential `n"
 	$usage += "Set-AvaDomain -domain corp.avanade.org -credential `$cred`n"
 	if ($help) {Write-Host $usage;exit}
 	
 	$username = $credential.GetNetworkCredential().UserName
 	$password = $credential.GetNetworkCredential().Password
 	$computer = Get-WmiObject Win32_ComputerSystem
 	$computer.JoinDomainOrWorkGroup($domain ,$password, $username, $null, 3)
 	
 	}
`

