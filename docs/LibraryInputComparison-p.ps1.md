---
Author: lee holmes
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2191
Published Date: 2011-09-09t21
Archived Date: 2016-03-19t02
---

# libraryinputcomparison.p - 

## Description

from windows powershell cookbook (o’reilly) by lee holmes

## Comments



## Usage



## TODO



## function

`get-inputwithforeach`

## Code

`#
 #
 
 Set-StrictMode -Version Latest
 
 function Get-InputWithForeach($identifier)
 {
     Write-Host "Beginning InputWithForeach (ID: $identifier)"
 
     foreach($element in $input)
     {
         Write-Host "Processing element $element (ID: $identifier)"
         $element
     }
 
     Write-Host "Ending InputWithForeach (ID: $identifier)"
 }
 
 function Get-InputWithKeyword($identifier)
 {
     begin
     {
         Write-Host "Beginning InputWithKeyword (ID: $identifier)"
     }
 
     process
     {
         Write-Host "Processing element $_ (ID: $identifier)"
         $_
     }
 
     end
     {
         Write-Host "Ending InputWithKeyword (ID: $identifier)"
     }
 }
`

