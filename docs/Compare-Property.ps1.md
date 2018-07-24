---
Author: lee holmes
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2132
Published Date: 2011-09-09t21
Archived Date: 2016-05-17t12
---

# compare-property.ps1 - 

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
 
 Compare the property you provide against the input supplied to the script.
 This provides the functionality of simple Where-Object comparisons without
 the syntax required for that cmdlet.
 
 .EXAMPLE
 
 Get-Process | Compare-Property Handles gt 1000
 
 .EXAMPLE
 
 PS >Set-Alias ?? Compare-Property
 PS >dir | ?? PsIsContainer
 
 #>
 
 param(
     $Property,
 
     $Operator = "eq",
 
     $MatchText = "$true"
 )
 
 Begin { $expression = "`$_.$property -$operator `"$matchText`"" }
 Process { if(Invoke-Expression $expression) { $_ } }
`

