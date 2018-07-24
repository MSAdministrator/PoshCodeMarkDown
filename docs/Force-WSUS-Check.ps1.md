---
Author: st3v3o
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 4385
Published Date: 2015-08-12t18
Archived Date: 2015-12-08t19
---

# force wsus check - 

## Description

remotely force wsus check on servers within your network.  powershell remoting must be enabled.

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 
 Import-Module ActiveDirectory
 
 $comps = Get-ADComputer -Filter {operatingsystem -like "*server*"}
 
 $cred = Get-Credential
 
 Foreach ($comp in $comps) {
 
 Invoke-Command -computername $comp.Name -credential $cred { wuauclt.exe /detectnow }
 Write-Host Forced WSUS Check-In on $comp.Name
 
 }
`

