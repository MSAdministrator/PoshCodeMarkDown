---
Author: lee holmes
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2185
Published Date: 2011-09-09t21
Archived Date: 2016-05-17t10
---

# invoke-scriptblock.ps1 - 

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
 
 Apply the given mapping command to each element of the input. (Note that
 PowerShell includes this command natively, and calls it Foreach-Object)
 
 .EXAMPLE
 
 1,2,3 | Invoke-ScriptBlock { $_ * 2 }
 
 #>
 
 param(
     [ScriptBlock] $MapCommand
 )
 
 begin
 {
     Set-StrictMode -Version Latest
 }
 process
 {
     & $mapCommand
 }
`

