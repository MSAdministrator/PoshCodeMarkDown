---
Author: lee holmes
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2192
Published Date: 2011-09-09t21
Archived Date: 2016-03-19t00
---

# libraryinvocation.ps1 - 

## Description

from windows powershell cookbook (o’reilly) by lee holmes

## Comments



## Usage



## TODO



## script

`get-scriptname`

## Code

`#
 #
 
 Set-StrictMode -Version Latest
 
 
 function Get-ScriptName
 {
     $myInvocation.ScriptName
 }
 
 function Get-ScriptPath
 {
     Split-Path $myInvocation.ScriptName
 }
`

