---
Author: ross j micheals
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5573
Published Date: 2014-11-06t16
Archived Date: 2014-11-17t22
---

# flatten hashtable - 

## Description

flattens a hashtable, removing ‘unnecessary’ levels of nesting.

## Comments



## Usage



## TODO



## script

`convertto-flatterhashtable`

## Code

`#
 #
 <#
 
 .SYNOPSIS
     Flattens a hashtable, removing 'unnecessary' levels of nesting.
 
 
 .DESCRIPTION
 
     This command takes as input, a (Powershell) hashtable and outputs a new hashtable
     in which 'unnecessary' levels of nesting are removed. A hashtable contains an 'unnecessary' level 
     of nesting if that hashtable either *is* or *contains* a hashtable with a single key-value pair 
     and that value is also a hashtable.
     
     The command is run in either 'parent biased' or 'child biased' mode. In 'parent biased' mode,
     the command will preserve the key of the parent-most level of a flattened hashtable. In 'child
     biased' mode, the command will preserve the key of the child-most level of a flattened hashtable.
     
     DETAILED EXAMPLE
     
     Consider the hashtable @{x=@{y=@{z=1}}}. This hashtable contains a doubly nested hashtable.
     The outer hashtable contributes to an unnecessary level of nesting because it has a single (key, value) 
     pair (x, {y=@{z=1}}) where the value, {y=@{z=1}}, is also a hashtable. The nested hashtable {y=@{z=1}} 
     similarly contributes 'unnecessary' nesting. The innermost hashtable @{z=1} does not contribute 
     to 'unnecessary' nesting; it does not contain another hashtable.
        
     In 'parent biased' mode, @{x=@{y=@{z=1}}} flattens to @{x=1}. In 'child biased' mode, it flattens 
     to @{z=1}.
 
 
 .PARAMETER Source
 
     Required. The hashtable to be flattened.
 
 
 .PARAMETER ChildBiased
 
     Optional. Flattens the source hashtable in 'child biased' mode. If this switch is absent, then then
     command defaults to 'parent biased' mode.
 
 .EXAMPLE
 
     Flatten a nested hashtable using the default ('parent biased') mode.
 
     PS C:\> @{a=@{b=1}} | ConvertTo-FlatterHashtable
 
     Name                           Value                                                                                                
     ----                           -----                                                                                                
     a                              1  
 
 
 .EXAMPLE
 
     Flatten a nested hashtable using 'child biased' mode.
 
     PS C:\> ConvertTo-FlatterHashtable -Source @{a=@{b=@{c=1}}} -ChildBiased
 
     Name                           Value                                                                                                
     ----                           -----                                                                                                
     c                              1  
     
     
     
 #>
 
 function ConvertTo-FlatterHashtable {
     [CmdletBinding()]
     param(
         [Parameter(Mandatory=$True, ValueFromPipeline=$True, ValueFromPipelinebyPropertyName=$True)] [Hashtable] $Source, 
         [Switch] $ChildBiased
     )
     
     BEGIN {}
  
     PROCESS {
 
         function ConvertTo-ChildBiasedFlatterHashtable([hashtable] $Source) {
             $ht = @{}
         
             foreach ($key in $Source.Keys) {
                 $value = $Source[$key]
                 if ($value -and $value.Count -eq 1 -and $value -as [Hashtable]) {
                     $ht = ConvertTo-ChildBiasedFlatterHashtable $value
                 } else {
                     $ht[$key] = $value
                 }
             }
             $ht
         } 
 
         function ConvertTo-ParentBiasedFlatterHashtable([hashtable] $Source) {
     
             function FlattenHashtable($start) {
                 if ($start -and $start -as [Hashtable] -and $start.Count -eq 1) {
                     FlattenHashtable $start[[string]($start.Keys[0])]
                 } else {
                     $start
                 }
             }
 
             $ht = @{}
             foreach ($key in $Source.Keys) {
                 $ht[$key] =  FlattenHashtable $Source[$key]
             }
             $ht
         }
 
         if ($ChildBiased) {
             ConvertTo-ChildBiasedFlatterHashtable $Source
         } else {
             ConvertTo-ParentBiasedFlatterHashtable $Source
         }
     }
 
     END {}
     
 }
`

