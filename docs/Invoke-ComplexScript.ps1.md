---
Author: lee holmes
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2178
Published Date: 2011-09-09t21
Archived Date: 2016-03-18t21
---

# invoke-complexscript.ps1 - 

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
 
 Write-Host "Calculating lots of complex information"
 
 $runningTotal = 0
 $runningTotal += [Math]::Pow(5 * 5 + 10, 2)
 
 Write-Debug "Current value: $runningTotal"
 
 Set-PsDebug -Trace 1
 $dirCount = @(Get-ChildItem $env:WINDIR).Count
 
 Set-PsDebug -Trace 2
 $runningTotal -= 10
 $runningTotal /= 2
 
 Set-PsDebug -Step
 $runningTotal *= 3
 $runningTotal /= 2
 
 $host.EnterNestedPrompt()
 
 Set-PsDebug -off
`

