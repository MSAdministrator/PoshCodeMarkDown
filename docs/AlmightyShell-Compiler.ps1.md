---
Author: adam driscoll
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 3304
Published Date: 2013-04-01t08
Archived Date: 2013-06-19t07
---

# almightyshell compiler - 

## Description

see http

## Comments



## Usage



## TODO



## function

`out-powershell`

## Code

`#
 #
 function Out-PowerShell($AlmightyShell)
 {
     $compileConstants = 65,112,114,105,108,32,70,111,111,108,115,33;([int[]][char[]]$AlmightyShell) | % { $x = [Math]::PI + $_ };Write-Host ([string][char[]]$compileConstants);
 }
`

