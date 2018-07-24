---
Author: karl prosser
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2512
Published Date: 2011-02-17t21
Archived Date: 2015-11-09t13
---

# where-in - 

## Description

where-in and where-propertyin are filters that allow to pass through pipeline object that are in a specified array/collection, or that have a property that is in an array or collection. they also can take a scriptblock that can be used to implement a comparision when the relationship isn’t exact. in that scriptblock the variable $__ is created to represent the item in the collection being compared with the pipeline $_ object. see examples embedded.

## Comments



## Usage



## TODO



## function

`where-in`

## Code

`#
 #
 function where-in {
 [cmdletbinding()]
 param (
 [parameter(mandatory = $true,position = 1)]
 [system.Collections.IEnumerable]$collection,
 [parameter(position = 2)]
 [scriptblock]$predicate ,
 [parameter(valuefrompipeline = $true)]
 $pipelineobject
 )
     process {
         if ($predicate) {
             foreach ($__ in $collection) {
                 if(&$predicate) { 
                     write-Output $pipelineobject
                     break;
                 }
             }
         }
         else {        
             if ($collection -contains $pipelineobject) {
             write-Output $pipelineobject }
         }
     }
 }
 set-alias ?in where-in
 
 $a = (1..10) , (1..10) | % { $_ }
 $a | Where-in (3,4,8)
 
 
 gps | where-in  ("power","s")  { $_.processname.startswith($__) }
 
 function where-propertyin {
 [cmdletbinding()]
 param (
 [parameter(mandatory = $true,position = 1)]
 [system.Collections.IEnumerable]$collection,
 [parameter(mandatory = $true,position = 2)]
 [string] $propertyname,
 [parameter(position = 3)]
 [scriptblock]$predicate ,
 [parameter(valuefrompipeline = $true)]
 $pipelineobject
 )
     process {
         if ($predicate) {
             foreach ($__ in $collection) {
                 $_ = $pipelineobject.$propertyname
                 if(&$predicate) { 
                     write-Output $pipelineobject
                     break;
                 }
             }
         }
         else {        
             if ($collection -contains $pipelineobject.$propertyname) {
             write-Output $pipelineobject }
         }
     }
 }
 set-alias ?.in where-propertyin
 
 gps | where-propertyin ("powershell","svchost") processname 
 gps | where-propertyin ("power","s") processname { $_.startswith($__) }
`

