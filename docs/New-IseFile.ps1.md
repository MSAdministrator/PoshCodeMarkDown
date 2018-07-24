---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 1618
Published Date: 
Archived Date: 2010-02-03t16
---

# new-isefile - 

## Description

if you are using ise put this file anywhere into your path and functions depending on it can use it.

## Comments



## Usage



## TODO



## function

`new-isefile`

## Code

`#
 #
 function New-IseFile ($path = 'tmp_default.ps1')
 {
     $count   = $psise.CurrentPowerShellTab.Files.count
     $null    = $psIse.CurrentPowerShellTab.Files.Add()
     $Newfile = $psIse.CurrentPowerShellTab.Files[$count]
     $NewFile.SaveAs($path)
     $NewFile.Save([Text.Encoding]::default)
     $Newfile
 
 }
`

