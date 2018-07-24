---
Author: lee holmes
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2221
Published Date: 2011-09-09t21
Archived Date: 2016-05-17t13
---

# set-psbreakpointlasterro - 

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
 Set-StrictMode -Version Latest
 
 $lastError = $error[0]
 Set-PsBreakpoint $lastError.InvocationInfo.ScriptName `
     $lastError.InvocationInfo.ScriptLineNumber
`

