---
Author: lee holmes
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2137
Published Date: 2011-09-09t21
Archived Date: 2016-05-17t16
---

# copy-history.ps1 - 

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
 
 Copy selected commands from the history buffer into the clipboard as a script.
 
 .EXAMPLE
 
 Copy-History
 Copies the entire contents of the history buffer into the clipboard.
 
 .EXAMPLE
 
 Copy-History -5
 Copies the last five commands into the clipboard.
 
 .EXAMPLE
 
 Copy-History 2,5,8,4
 Copies commands 2,5,8, and 4.
 
 .EXAMPLE
 
 Copy-History (1..10+5+6)
 Copies commands 1 through 10, then 5, then 6, using PowerShell's array
 slicing syntax.
 
 #>
 
 param(
     [int[]] $Range
 )
 
 Set-StrictMode -Version Latest
 
 $history = @()
 
 if((-not $range) -or ($range.Count -eq 0))
 {
     $history = @(Get-History -Count ([Int16]::MaxValue))
 }
 elseif(($range.Count -eq 1) -and ($range[0] -lt 0))
 {
     $count = [Math]::Abs($range[0])
     $history = (Get-History -Count $count)
 }
 else
 {
     foreach($commandId in $range)
     {
         if($commandId -eq -1) { $history += Get-History -Count 1 }
         else { $history += Get-History -Id $commandId }
     }
 }
 
 $history | Foreach-Object { $_.CommandLine } | clip.exe
`

