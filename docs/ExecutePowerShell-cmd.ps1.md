---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2447
Published Date: 
Archived Date: 2011-01-10t08
---

# executepowershell.cmd - 

## Description

this is a batch file … with a powershell script inside.  it’s my answer to all those “compile your .ps1” solutions that are floating around. why would you do that, when powershell still has to be installed?

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 :: <#
 copy %0 %0.ps1
 PowerShell.exe -ExecutionPolicy Unrestricted -NoProfile -Command "$ErrorActionPreference = 'SilentlyContinue'; . %0.ps1; Remove-Item %0.ps1"
 exit
 :: #>
 $ErrorActionPreference = 'Continue'
 
 ls | sort length -desc | select -first 5 | ft
 ps | sort ws -desc | select -first 10 | ft
`

