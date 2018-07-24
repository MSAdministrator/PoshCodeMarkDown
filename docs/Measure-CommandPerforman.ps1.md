---
Author: lee holmes
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2195
Published Date: 2011-09-09t21
Archived Date: 2016-03-18t21
---

# measure-commandperforman - 

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
 ##############################################################################
 ##
 ##
 ##
 ##############################################################################
 
 <#
 
 .SYNOPSIS
 
 Measures the average time of a command, accounting for natural variability by
 automatically ignoring the top and bottom ten percent.
 
 .EXAMPLE
 
 PS >Measure-CommandPerformance.ps1 { Start-Sleep -m 300 }
 
 Count    : 30
 Average  : 312.10155
 (...)
 
 #>
 
 param(
     [Scriptblock] $Scriptblock,
 
     [int] $Iterations = 30
 )
 
 Set-StrictMode -Version Latest
 
 $buffer = [int] ($iterations * 0.1)
 $totalIterations = $iterations + (2 * $buffer)
 
 $results = 1..$totalIterations |
     Foreach-Object { Measure-Command $scriptblock }
 
 $middleResults = $results | Sort TotalMilliseconds |
     Select -Skip $buffer -First $iterations
 
 $middleResults | Measure-Object -Average TotalMilliseconds
`

