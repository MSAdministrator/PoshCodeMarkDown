---
Author: alphasun
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6296
Published Date: 2017-04-08t21
Archived Date: 2017-04-30t12
---

# bulk delprof - 

## Description

this script provides a way to bulk delete profiles on a system. utilizes the delprof2.exe program.

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 $Input = Read-Host	'Dot-sourced path to input file (eg .\Targets.txt)'
 Write-Host		' '
 Write-Host		'Wildcard characters * and ? can be used in the pattern'
 Write-Host		' '
 $Include = Read-Host	'Include profiles that match this naming pattern'
 $Exclude = Read-Host	'Exclude profiles that match this naming pattern'
 
 $Date = Get-Date -Format yyyyMMdd.hhmm
 $Targets = Get-Content $Input
 foreach ($target in $Targets) {
 
 }
 
 Write-Host -NoNewLine "Press any key to continue..."
 $null = $Host.UI.RawUI.ReadKey("NoEcho,IncludeKeyDown")
`

