---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 4968
Published Date: 2014-03-09t01
Archived Date: 2014-03-10t23
---

# convertto-hashtable - 

## Description

converts objects properties into key-value hashtable

## Comments



## Usage



## TODO



## function

`convertto-hashtable`

## Code

`#
 #
 function ConvertTo-Hashtable {
   #.Synopsis
   PARAM(
     [Parameter(ValueFromPipeline=$true, Mandatory=$true)]
     $InputObject,
 
     [switch]$AsString,  
 
     [switch]$jagged,
 
     [switch]$NoNulls
   )
   BEGIN { 
     $headers = @() 
   }
   PROCESS {
     if(!$headers -or $jagged) {
       $headers = $InputObject | get-member -type Properties | select -expand name
     }
     $output = @{}
     if($AsString) {
       foreach($col in $headers) {
         if(!$NoNulls -or ($InputObject.$col -is [bool] -or ($InputObject.$col))) {
           $output.$col = $InputObject.$col | out-string -Width 9999 | % { $_.Trim() }
         }
       }
     } else {
       foreach($col in $headers) {
         if(!$NoNulls -or ($InputObject.$col -is [bool] -or ($InputObject.$col))) {
           $output.$col = $InputObject.$col
         }
       }
     }
     $output
   }
 }
`

