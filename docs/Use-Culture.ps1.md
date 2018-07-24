---
Author: lee holmes
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6449
Published Date: 2016-07-16t16
Archived Date: 2016-07-20t02
---

# use-culture.ps1 - 

## Description

from windows powershell cookbook (o’reilly) by lee holmes

## Comments



## Usage



## TODO



## script

`set-culture`

## Code

`#
 #
 #############################################################################
 ##
 ##
 ##
 #############################################################################
 
 <#
 
 .SYNOPSIS
 
 Invoke a scriptblock under the given culture
 
 .EXAMPLE
 
 Use-Culture fr-FR { [DateTime]::Parse("25/12/2007") }
 mardi 25 decembre 2007 00:00:00
 
 #>
 
 param(
     [Parameter(Mandatory = $true)]
     [System.Globalization.CultureInfo] $Culture,
 
     [Parameter(Mandatory = $true)]
     [ScriptBlock] $ScriptBlock
 )
 
 Set-StrictMode -Version Latest
 
 function Set-Culture([System.Globalization.CultureInfo] $culture)
 {
     [System.Threading.Thread]::CurrentThread.CurrentUICulture = $culture
     [System.Threading.Thread]::CurrentThread.CurrentCulture = $culture
 }
 
 $oldCulture = [System.Threading.Thread]::CurrentThread.CurrentUICulture
 
 trap { Set-Culture $oldCulture }
 
 Set-Culture $culture
 
 & $ScriptBlock
 
 Set-Culture $oldCulture
`

