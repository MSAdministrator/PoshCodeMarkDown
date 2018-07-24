---
Author: lee holmes
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2183
Published Date: 2011-09-09t21
Archived Date: 2016-03-18t21
---

# invoke-member.ps1 - 

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
 
 Enables easy access to methods and properties of pipeline objects.
 
 .EXAMPLE
 
 PS >"Hello","World" | .\Invoke-Member Length
 5
 5
 
 .EXAMPLE
 
 PS >"Hello","World" | .\Invoke-Member -m ToUpper
 HELLO
 WORLD
 
 .EXAMPLE
 
 PS >"Hello","World" | .\Invoke-Member Replace l w
 Hewwo
 Worwd
 
 #>
 
 [CmdletBinding(DefaultParameterSetName= "Member")]
 param(
 
     [Parameter(ParameterSetName = "Method")]
     [Alias("M","Me")]
     [switch] $Method,
 
     [Parameter(ParameterSetName = "Method", Position = 0)]
     [Parameter(ParameterSetName = "Member", Position = 0)]
     [string] $Member,
 
     [Parameter(
         ParameterSetName = "Method", Position = 1,
         Mandatory = $false, ValueFromRemainingArguments = $true)]
     [object[]] $ArgumentList = @(),
 
     [Parameter(ValueFromPipeline = $true)]
     $InputObject
     )
 
 begin
 {
     Set-StrictMode -Version Latest
 }
 
 process
 {
     if($psCmdlet.ParameterSetName -eq "Method")
     {
         $inputObject.$member.Invoke(@($argumentList))
     }
     else
     {
         $inputObject.$member
     }
 }
`

