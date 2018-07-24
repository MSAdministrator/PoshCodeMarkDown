---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 1617
Published Date: 
Archived Date: 2010-02-03t16
---

# require-function - 

## Description

this is a bit similar to python import module. if a function is present, nothing happens, otherwise path is searched for a file with the functionname and the extension .ps1 and that file is dot sourced

## Comments



## Usage



## TODO



## function

`require-function`

## Code

`#
 #
 function Require-Function
 {
 .SYNOPSIS 
     Load function when not already loaded    
 .DESCRIPTION 
     When a function is not loaded and there is a file <functionname>.ps1 in one of the directories listed
     in $env:Path it is dot sourced.
     To get the function in your environment you must dot source the Require-Function too.
 .NOTES 
     File Name  : Require-Function.ps1 
     Author     : Bernd Kriszio - http://pauerschell.blogspot.com/ 
 .PARAMETER FunctionName
     The name of the function you want to import
 .EXAMPLE 
     . Require-Function <any_function_script_in_your_path>
 .EXAMPLE 
     . Require-Function New-ISEFile -verbose
 .EXAMPLE 
     . Require-Function New-ISEFile
 .EXAMPLE 
    . Require-Function unbekanntn -verbose
 #> 
    [cmdletbinding()]
    param (
     [string] $FunctionName
     )
     
     if (! (Test-Path function:$FunctionName))
     {
         try{
             Write-Verbose "Function $FunctionName not loaded"
             $cmd = "$($FunctionName).ps1"
             #"$cmd" 
             .  $cmd
         }
         catch
         {
             "$cmd could not be loaded"
             break;
         }
         Write-Verbose "Function $FunctionName is loaded"
     }
     else
     {
         Write-Verbose "Function $FunctionName was loaded"
     }
 }
`

