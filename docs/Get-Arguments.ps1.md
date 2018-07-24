---
Author: lee holmes
Publisher: 
Copyright: 
Email: 
Version: 4.5
Encoding: ascii
License: cc0
PoshCode ID: 2148
Published Date: 2011-09-09t21
Archived Date: 2016-05-17t13
---

# get-arguments.ps1 - 

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
 
 Uses command-line arguments
 
 #>
 
 param(
     $FirstNamedArgument,
 
     [int] $SecondNamedArgument = 0
 )
 
 Set-StrictMode -Version Latest
 
 "First named argument is: $firstNamedArgument"
 "Second named argument is: $secondNamedArgument"
 
 function GetArgumentsFunction
 {
 
     "First positional function argument is: " + $args[0]
     "Second positional function argument is: " + $args[1]
 }
 
 GetArgumentsFunction One Two
 
 $scriptBlock =
 {
     param($firstNamedArgument, [int] $secondNamedArgument = 0)
 
     "First named scriptblock argument is: $firstNamedArgument"
     "Second named scriptblock argument is: $secondNamedArgument"
 }
 
 & $scriptBlock -First One -Second 4.5
`

