---
Author: lee holmes
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5972
Published Date: 2016-08-11t08
Archived Date: 2016-05-24t22
---

# get-clipboard.ps1 - 

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
 #############################################################################
 ##
 ##
 ##
 ##############################################################################
 
 <#
 
 .SYNOPSIS
 
 Retrieve the text contents of the Windows Clipboard.
 
 .EXAMPLE
 
 PS >Get-Clipboard
 Hello World
 
 #>
 
 Set-StrictMode -Version Latest
 
 PowerShell -NoProfile -STA -Command {
     Add-Type -Assembly PresentationCore
     [Windows.Clipboard]::GetText()
 }
`

