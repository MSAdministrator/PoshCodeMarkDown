---
Author: lee holmes
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2224
Published Date: 2011-09-09t21
Archived Date: 2016-03-18t22
---

# show-htmlhelp.ps1 - 

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
 
 Launches the CHM version of PowerShell help.
 
 .EXAMPLE
 
 Show-HtmlHelp
 
 #>
 
 Set-StrictMode -Version Latest
 
 $path = (Resolve-Path c:\windows\help\mui\*\WindowsPowerShellHelp.chm).Path
 hh "$path::/html/defed09e-2acd-4042-bd22-ce4bf92c2f24.htm"
`

