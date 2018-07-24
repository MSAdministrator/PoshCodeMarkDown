---
Author: lee holmes
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2215
Published Date: 2011-09-09t21
Archived Date: 2016-03-18t22
---

# select-textoutput.ps1 - 

## Description

from windows powershell cookbook (o’reilly) by lee holmes

## Comments



## Usage



## TODO



## 

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
 
 Searches the textual output of a command for a pattern.
 
 .EXAMPLE
 
 Get-Service | Select-TextOutput audio
 Finds all references to "Audio" in the output of Get-Service
 
 #>
 
 param(
     $Pattern
 )
 
 Set-StrictMode -Version Latest
 $input | Out-String -Stream | Select-String $pattern
`

