---
Author: oisin grehan
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5515
Published Date: 2015-10-14t18
Archived Date: 2015-07-24t21
---

# invoke-anything - 

## Description

late bound invocation for any methods/generic methods (or properties), including open generic members that would normally require awkward reflection and near-impossible manual generic type parameter inference. pipe the type or instance, pass the method name as a string, the arguments as an object array and don’t forget to tell it if you want to call a static method.

## Comments



## Usage



## TODO



## function

`invoke-lateboundmember`

## Code

`#
 #
 function Invoke-LateBoundMember {
     [cmdletbinding(supportsshouldprocess)]
     param(
         [parameter(valuefrompipeline, mandatory)]
         [validatenotnull()]
         [psobject[]]$InputObject,
 
         [parameter(mandatory, position=0)]
         [validatenotnullorempty()]
         [string]$MemberName,
 
         [parameter(position=1, valuefromremainingarguments)]
         [object[]]$ArgumentList,
 
         [parameter()]
         [switch]$Static,
 
         [parameter()]
         [switch]$IgnoreReturn
     )
 
     begin {
         add-type -AssemblyName Microsoft.VisualBasic
         $vbbinder = [Microsoft.VisualBasic.CompilerServices.NewLateBinding]
     }
     
     process {
 
         foreach ($element in $InputObject) {
 
             $instance = $null
             $type = $null
             if ($Static) { $type = $element } else { $instance = $element }
 
             $copyBack = new-object bool[] $ArgumentList.Length
             
 
             #$unwrapped = $argumentlist | % { if ($_ -is [ref]) { $_.value } else { $_ } }
             
             $unwrapped = new-object object[] $ArgumentList.Count
             
             if ($ArgumentList) {
                 [array]::Copy($ArgumentList, $unwrapped, $ArgumentList.length)
                 for ($i = 0; $i -lt $ArgumentList.Length; $i++) {
                     if ($unwrapped[$i] -is [ref]) { $unwrapped[$i] = $unwrapped[$i].value }
                     if ($unwrapped[$i] -is [psobject]) {
                         write-warning "The argument at index $i is wrapped as a PSObject. This is almost certainly not what you want."
                     }
                 }
             }
 
             $result = $vbbinder::LateCall(
                 [string]$MemberName,
                 $unwrapped,
 
             for ($i = 0; $i -lt $ArgumentList.Length; $i++) {
                 $argument = $ArgumentList[$i]
                 if ($copyBack[$i]) {
                     if ($argument -is [ref]) {
                         write-verbose "Updating [ref] at index $i"
                         $argument.value = $unwrapped[$i]
                     } else {
                         write-warning "Argument at index $i was modified. Pass the argument as a [ref] variable to receive the result."
                     }
                 }
             }
 
             if (-not $IgnoreReturn) {
                 $result
             }
         }
     }
 }
 
 New-Alias -Name ilb -Value Invoke-LateBoundMember -Force
`

