---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5135
Published Date: 
Archived Date: 2014-05-05t02
---

#  - 

## Description

trims text and signs it

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 (Get-Content c:\loltweaks.export.ps1) | ` Where-Object { $_ -match '\S' } | ` Out-File c:\loltweaks.ps1
 $filename = "c:\loltweaks.ps1"
 $lines = (Get-Content $filename);
 $lines | ForEach-Object { $_.Trim(); } | Out-File C:\LoLtweaks.ps1
 Set-AuthenticodeSignature C:\LoLtweaks.ps1 @(Get-ChildItem cert:\CurrentUser\My -codesigning)[0] -TimestampServer http://timestamp.comodoca.com/authenticode
`

