---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 1132
Published Date: 
Archived Date: 2009-05-31t07
---

# get-snmphost.ps1 - 

## Description

gets the vmhostsnmp object for vmware vi toolkit consumption

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 Param($VC,$ESXCreds=(Get-Credential))
 
 Write-Host "Connecting to VC to get ESX Hosts"
 Connect-VIServer $VC | out-null
 
 $ESXHosts = Get-VMHost
 
 foreach($esxhost in $ESXHosts)
 {
    Write-Host " Connecting to $esxhost"
    Connect-VIServer $esxhost.name -cred $ESXCreds | out-null
    $SNMPHost = Get-VMHostSnmp
    $SNMPHost | Add-Member -MemberType NoteProperty -Name ESXHost -Value $esxhost.name
    $SNMPHost
 }
`

