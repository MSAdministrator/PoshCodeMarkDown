---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 4960
Published Date: 
Archived Date: 2014-03-07t19
---

#  - 

## Description

$check.installed not refreshed after feature instlled

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 $check = Get-WindowsFeature | Where-Object {$_.Name -eq "SNMP-Service"}
 
 If ($check.Installed -ne "True") {
 	Add-WindowsFeature SNMP-Service 
 }
 
 $check = Get-WindowsFeature | Where-Object {$_.Name -eq "SNMP-Service"}
`

