---
Author: nathan hartley
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 546
Published Date: 2009-08-22t14
Archived Date: 2014-03-12t00
---

# get-oucomputernames - 

## Description

usage

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 param( 
 	[switch]$Help,
 	[string] $OU
 	)
 $usage = @'
 Get-OUComputerNames
 usage   : .\Get-OUComputerNames.ps1 "OU=TESTOU,dc=domain,dc=com"
 returns : the names of computers in Active Directory (AD) that
           recursively match a given LDAP query
 author  : Nathan Hartley
 '@
 if ($help) {Write-Host $usage;break}
 if (!$OU) {$OU = $(read-host "Please specify an LDAP search string")}
 if (!$OU -match "^LDAP://") { $OU = "LDAP://" + $OU }
 
 $strCategory = "computer"
 
 $objDomain = New-Object System.DirectoryServices.DirectoryEntry("LDAP://$OU")
 $objSearcher = New-Object System.DirectoryServices.DirectorySearcher($objDomain,"(objectCategory=$strCategory)",@('name'))
 $objSearcher.FindAll() | %{$_.properties.name}
`

