---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 1336
Published Date: 
Archived Date: 2009-09-28t13
---

# set-ft - 

## Description

script used to turn ft on or off for a specific vm

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 param([string]$ftVM, [string]$ftState)
 
 add-pssnapin VMware.VimAutomation.Core
 
 $creds = Get-VICredentialStoreItem -file c:\test 
 
 connect-viserver -Server $creds.Host -User $creds.User -Password $creds.Password
 
 $ftState = $ftState.ToLower()
 if ($ftState -eq "on"){
 	Get-VM X | Get-View | % { $_.CreateSecondaryVM($null) }
 } elseif ($ftstate -eq "off"){
 	Get-VM X | Select -First 1 | Get-View | % { $_.TurnOffFaultToleranceForVM() }
 } else {
 	Write-Error "FT State must be either On or Off"
 }
`

