---
Author: oisin grehan
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 3661
Published Date: 2013-09-24t13
Archived Date: 2016-03-17t04
---

# convertto-module - 

## Description

quickly convert a .net type’s static methods into functions. (bugfix update

## Comments



## Usage



## TODO



## function

`convertto-module`

## Code

`#
 #
 function ConvertTo-Module {
 <#
     .SYNOPSIS
     Quickly convert a .NET type's static methods into functions
 
     .DESCRIPTION
     Quickly convert a .NET type's static methods into functions.
     
     This function returns a PSModuleInfo, so you should pipe its
     output to Import-Module to use the exported functions.
 
     .PARAMETER Type
     The type from which to import static methods. 
 
     .INPUTS
     System.String, System.Type
 
     .OUTPUTS
     PSModuleInfo
 
     .EXAMPLE
     ConvertTo-Module System.Math | Import-Module -Verbose
 
     .EXAMPLE
     [math] | ConvertTo-Module | Import-Module -Verbose
 
 #>
     [outputtype([psmoduleinfo])]
     param(
         [parameter(
             position=0,
             valuefrompipeline=$true,
             mandatory=$true)]
         [validatenotnull()]
         [type]$Type
     )
 
     new-module {
         param($type)
          
         ($exports = $type.getmethods("static,public").Name | sort -uniq) | `
             % {
                 $func = $_
                 new-item "function:script:$($_)" `
                     -Value {
                         ($type.FullName -as [type])::$func.invoke($args)
 
             }
         export-modulemember -function $exports
     } -name $type.Name -ArgumentList $type
 }
`

