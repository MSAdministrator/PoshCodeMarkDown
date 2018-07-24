---
Author: lee holmes
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2186
Published Date: 2011-09-09t21
Archived Date: 2016-03-19t00
---

# invoke-scriptblockclosur - 

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
 
 Demonstrates the GetNewClosure() method on a script block that pulls variables
 in from the user's session (if they are defined.)
 
 .EXAMPLE
 
 PS >$name = "Hello There"
 PS >Invoke-ScriptBlockClosure { $name }
 Hello There
 Hello World
 Hello There
 
 #>
 
 param(
     [ScriptBlock] $ScriptBlock
 )
 
 Set-StrictMode -Version Latest
 
 $closedScriptBlock = $scriptBlock.GetNewClosure()
 
 & $scriptBlock
 
 $name = "Hello World"
 
 & $scriptBlock
 
 & $closedScriptBlock
`

