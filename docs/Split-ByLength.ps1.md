---
Author: dang_it_bobby
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 4205
Published Date: 2013-06-18t17
Archived Date: 2016-08-27t11
---

# split-bylength - 

## Description

http

## Comments



## Usage



## TODO



## function

`split-bylength`

## Code

`#
 #
 function Split-ByLength{
     <#
     .SYNOPSIS
     Splits string up by Split length.
 
     .DESCRIPTION
     Convert a string with a varying length of characters to an array formatted to a specific number of characters per item.
 
     .EXAMPLE
     Split-ByLength '012345678901234567890123456789123' -Split 10
 
     0123456789
     0123456789
     0123456789
     123
 
     .LINK
     #>
 
     [cmdletbinding()]
     param(
         [Parameter(ValueFromPipeline=$true)]
         [string[]]$InputObject,
 
         [int]$Split=10
     )
     begin{}
     process{
         foreach($string in $InputObject){
             $len = $string.Length
 
             $repeat=[Math]::Floor($len/$Split)
 
             for($i=0;$i-lt$repeat;$i++){
                 Write-Output $string.Substring($i*$Split,$Split)
             }
             if($remainder=$len%$split){
                 Write-Output $string.Substring($len-$remainder)
             }
         }        
     }
     end{}
 }
`

