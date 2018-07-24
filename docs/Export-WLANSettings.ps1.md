---
Author: jan egil ring
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6226
Published Date: 2016-02-19t19
Archived Date: 2016-03-18t21
---

# export-wlansettings.ps1 - 

## Description

using netsh.exe to loop through each wlan on the system and export the settings to the user-provided output-file.

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
  #
  #
 
 Write-Warning "Must be run with Administrator-privileges for Key Content to be exported"
 $filepath = Read-Host "Provide path to output-file. E.g. C:\temp\wlan.txt"
 
 $wlans = netsh wlan show profiles | Select-String -Pattern "All User Profile" | Foreach-Object {$_.ToString()}
 $exportdata = $wlans | Foreach-Object {$_.Replace("    All User Profile     : ",$null)}
 $exportdata | ForEach-Object {netsh wlan show profiles name="$_" key=clear} | Out-File $filepath
`

