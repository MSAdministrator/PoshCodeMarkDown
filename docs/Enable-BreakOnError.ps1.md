---
Author: lee holmes
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2138
Published Date: 2011-09-09t21
Archived Date: 2016-05-17t12
---

# enable-breakonerror.ps1 - 

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
 #############################################################################
 ##
 ##
 ##
 ##############################################################################
 
 <#
 
 .SYNOPSIS
 
 Creates a breakpoint that only fires when PowerShell encounters an error
 
 .EXAMPLE
 
 PS >Enable-BreakOnError
 
 ID Script           Line Command         Variable        Action
 -- ------           ---- -------         --------        ------
  0                       Out-Default                     ...
 
 PS >1/0
 Entering debug mode. Use h or ? for help.
 
 Hit Command breakpoint on 'Out-Default'
 
 
 PS >$error
 Attempted to divide by zero.
 
 #>
 
 Set-StrictMode -Version Latest
 
 $GLOBAL:EnableBreakOnErrorLastErrorCount = $error.Count
 
 Set-PSBreakpoint -Command Out-Default -Action {
 
     if($error.Count -ne $EnableBreakOnErrorLastErrorCount)
     {
         $GLOBAL:EnableBreakOnErrorLastErrorCount = $error.Count
         break
     }
 }
`

