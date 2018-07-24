---
Author: walter mitty
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5915
Published Date: 2015-07-01t12
Archived Date: 2015-07-03t01
---

# templates and tables - 

## Description

this script generates an output file from a template and a driver table.  both of the inputs are files. the template file contains plain

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
     It's a refinement of an earlier attempt.
 
     It generates an output file from a template and
     a driver table.  The template file contains plain
     text and embedded variables.  The driver table has
     one column for each variable, and one row for each
     expansion to be generated.
 
     2/15/2015
   
 #>
 
 param ($driver, $template, $out);
 
 $OFS = "`r`n"
 $list = Import-Csv $driver
 [string]$pattern = Get-Content $template
 Clear-Content $out -ErrorAction SilentlyContinue
 
 foreach ($item in $list) {
    foreach ($key in $item.psobject.properties) {
       Set-variable -name $key.name -value $key.value
       }
    $ExecutionContext.InvokeCommand.ExpandString($pattern)  >> $out
    }
`

