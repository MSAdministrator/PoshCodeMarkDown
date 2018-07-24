---
Author: chad miller
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 1986
Published Date: 2011-07-19t19
Archived Date: 2015-06-27t05
---

# converttostringdata - 

## Description

converts a hashtable to a string representation of the hashtable definition. useful in persisting hashtables to .net isolated storage

## Comments



## Usage



## TODO



## function

`convertto-stringdata`

## Code

`#
 #
 function ConvertTo-StringData
 { 
     Begin 
     { 
        $string  = "@{`n"
         function Expand-Value
         {
             param($value)
 
             if ($value -ne $null) {
                 switch ($value.GetType().Name)
                 {
                     'String' { "`"$value`"" }
                     'Boolean' { "`$$value" }
                     default { $value }
                 }
             }
             else
             { "`$null" }
 
         }
     } 
     Process 
     { 
         $string += $_.GetEnumerator() | foreach {"{0} = {1}`n" -f $_.Name,(Expand-Value $_.Value)}
     } 
     End 
     { 
         $string += "}" 
         Write-Output $string
     }
 
`

