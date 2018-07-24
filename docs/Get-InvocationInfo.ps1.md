---
Author: lee holmes
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2156
Published Date: 2011-09-09t21
Archived Date: 2016-03-18t21
---

# get-invocationinfo.ps1 - 

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
 
 Display the information provided by the $myInvocation variable
 
 #>
 
 param(
     [switch] $PreventExpansion
 )
 
 Set-StrictMode -Version Latest
 
 function HelperFunction
 {
     "    MyInvocation from function:"
     "-"*50
     $myInvocation
 
     "    Command from function:"
     "-"*50
     $myInvocation.MyCommand
 }
 
 $myScriptBlock = {
     "    MyInvocation from script block:"
     "-"*50
     $myInvocation
 
     "    Command from script block:"
     "-"*50
     $myInvocation.MyCommand
 }
 
 Set-Alias gii .\Get-InvocationInfo
 
 "You invoked this script by typing: " + $myInvocation.Line
 
 "MyInvocation from script:"
 "-"*50
 $myInvocation
 
 "Command from script:"
 "-"*50
 $myInvocation.MyCommand
 
 if($preventExpansion)
 {
     return
 }
 
 "Calling HelperFunction"
 "-"*50
 HelperFunction
 
 "Dot-Sourcing HelperFunction"
 "-"*50
 . HelperFunction
 
 "Calling aliased script"
 "-"*50
 gii -PreventExpansion
 
 "Calling script block"
 "-"*50
 & $myScriptBlock
 
 "Dot-Sourcing script block"
 "-"*50
 . $myScriptBlock
 
 "Calling aliased script"
 "-"*50
 gii -PreventExpansion
`

