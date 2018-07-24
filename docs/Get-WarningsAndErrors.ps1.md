---
Author: lee holmes
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2168
Published Date: 2011-09-09t21
Archived Date: 2016-03-18t21
---

# get-warningsanderrors.ps - 

## Description

from windows powershell cookbook (o’reilly) by lee holmes

## Comments



## Usage



## TODO



## function

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
 
 Demonstrates the functionality of the Write-Warning, Write-Error, and throw
 statements
 
 #>
 
 Set-StrictMode -Version Latest
 
 Write-Warning "Warning: About to generate an error"
 Write-Error "Error: You are running this script"
 throw "Could not complete operation."
`

