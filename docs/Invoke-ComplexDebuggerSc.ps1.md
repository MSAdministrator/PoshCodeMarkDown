---
Author: lee holmes
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2177
Published Date: 2011-09-09t21
Archived Date: 2016-03-18t23
---

# invoke-complexdebuggersc - 

## Description

from windows powershell cookbook (o’reilly) by lee holmes

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 #############################################################################
 ##
 ##
 ##
 ##############################################################################
 
 <#
 
 .SYNOPSIS
 
 Demonstrates the functionality of PowerShell's debugging support.
 
 #>
 
 Set-StrictMode -Version Latest
 
 function HelperFunction
 {
     $dirCount = 0
 }
 
 Write-Host "Calculating lots of complex information"
 
 $runningTotal = 0
 $runningTotal += [Math]::Pow(5 * 5 + 10, 2)
 $runningTotal
 
 $dirCount = @(Get-ChildItem $env:WINDIR).Count
 $dirCount
 
 HelperFunction
 
 $dirCount
 
 $runningTotal -= 10
 $runningTotal /= 2
 $runningTotal
 
 $runningTotal *= 3
 $runningTotal /= 2
 $runningTotal
`

