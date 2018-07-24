---
Author: adam mendoza
Publisher: 
Copyright: 
Email: 
Version: 3.0
Encoding: ascii
License: cc0
PoshCode ID: 3473
Published Date: 2013-06-22t14
Archived Date: 2016-06-19t06
---

# check powershell version - 

## Description

check if powershell version 3 or higher is installed

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 
 if($host.Version.Major -lt 3)
 {
  Write-Host "PowerShell Version 3 or higher needs to be installed"  -ForegroundColor Red
  Write-Host "Windows Management Framework 3.0 - RC"  -ForegroundColor Magenta
  Write-Host "http://www.microsoft.com/en-us/download/details.aspx?id=29939"  -ForegroundColor Magenta
  Break
 }
`

