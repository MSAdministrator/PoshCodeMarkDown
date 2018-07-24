---
Author: lee holmes
Publisher: 
Copyright: 
Email: 
Version: 85.00
Encoding: ascii
License: cc0
PoshCode ID: 2144
Published Date: 2011-09-09t21
Archived Date: 2016-05-17t10
---

# format-string.ps1 - 

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
 
 Replaces text in a string based on named replacement tags
 
 .EXAMPLE
 
 Format-String "Hello {NAME}" @{ NAME = 'PowerShell' }
 Hello PowerShell
 
 .EXAMPLE
 
 Format-String "Your score is {SCORE:P}" @{ SCORE = 0.85 }
 Your score is 85.00 %
 
 #>
 
 param(
     $String,
 
     [hashtable] $Replacements
 )
 
 Set-StrictMode -Version Latest
 
 $currentIndex = 0
 $replacementList = @()
 
 foreach($key in $replacements.Keys)
 {
     $inputPattern = '{(.*)' + $key + '(.*)}'
     $replacementPattern = '{${1}' + $currentIndex + '${2}}'
     $string = $string -replace $inputPattern,$replacementPattern
     $replacementList += $replacements[$key]
 
     $currentIndex++
 }
 
 $string -f $replacementList
`

