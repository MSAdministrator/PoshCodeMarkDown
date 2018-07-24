---
Author: bsonposh
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6437
Published Date: 2016-07-01t11
Archived Date: 2016-10-15t17
---

# whitebox - 

## Description

it gets the system environment variables from registry

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 Param($srv=$env:ComputerName)
 $regKey = [Microsoft.Win32.RegistryKey]::OpenRemoteBaseKey([Microsoft.Win32.RegistryHive]::LocalMachine,$Srv)
 $key = $regKey.OpenSubkey("SYSTEM\CurrentControlSet\Control\Session Manager\Environment",$false)
 $key.GetValueNames() | Select-Object @{n="ValueName";e={$_}},@{n="Value";e={$key.GetValue($_)}}
`

