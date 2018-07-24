---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2750
Published Date: 2011-06-24t16
Archived Date: 2011-11-17t12
---

# new-tiedvariable - 

## Description

a function for creating tied variables using robert robelo’s idea to create breakpoints that update the variable values.

## Comments



## Usage



## TODO



## function

`new-tiedvariable`

## Code

`#
 #
 function New-TiedVariable {
 #.Synopsis
 #.Description
 #.Notes
 param(
     [String]$Name,
     [ScriptBlock]$Value
 )
     Set-Variable $Name -Value (.$Value) -Option ReadOnly, AllScope -Scope Global -Force
 
     $null = Set-PSBreakpoint -Variable $Name -Mode Read -Action {
         Set-Variable $Name (. $Value) -Option ReadOnly, AllScope -Scope Global -Force
     }.GetNewClosure()
 }
`

