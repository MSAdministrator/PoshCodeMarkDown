---
Author: oisin grehan
Publisher: 
Copyright: 
Email: 
Version: 2.0
Encoding: ascii
License: cc0
PoshCode ID: 1773
Published Date: 
Archived Date: 2010-04-17t07
---

# push-module function v2 - 

## Description

push-module will import a powershell v2.0 module and ensure that when the module is removed using remove-module, the functions the module clobbered are restored. update

## Comments



## Usage



## TODO



## function

`push-module`

## Code

`#
 #
 
 function Push-Module {
     param(
         [parameter(position=0, mandatory=$true)]
         [validatenotnullorempty()]
         [string]$ModuleName
     )
     
     $metadata = new-module -ascustomobject {
         param([string]$ModuleName)
         
         $inner = import-module $ModuleName -PassThru       
         function GetExportedFunctions() {
             $inner.ExportedFunctions.values
         }
         
         Export-ModuleMember -Cmdlet "" -Function GetExportedFunctions
         
     } -args $ModuleName
         
     $test = new-module -ascustomobject {
         param($metadata)
         
         $clobbered = @{}
         
         $metadata.GetExportedFunctions() | ? {
             Test-Path "function:$($_.name)"
         } | % {    
             $clobbered[$_.name] = gc -path "function:$($_.name)"
         }
         
         function GetClobberedFunctions() {
             $clobbered
         }
         
     } -args $metadata
 
     $m = import-module $ModuleName -PassThru
 
     $clobbered = $test.GetClobberedFunctions()
         
     $m.onremove = {
         $clobbered.keys | % {
             si -path function:global:$_ $clobbered[$_] -force
         }
     }.getnewclosure()
 }
`

