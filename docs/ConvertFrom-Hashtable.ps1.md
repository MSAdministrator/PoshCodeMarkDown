---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 1118
Published Date: 2010-05-20t07
Archived Date: 2017-03-03t08
---

# convertfrom-hashtable - 

## Description

this script has appeared in many places in many different forms. eg

## Comments



## Usage



## TODO



## function

`convertfrom-hashtable`

## Code

`#
 #
 PARAM([HashTable]$hashtable,[switch]$combine)
 BEGIN { $output = New-Object PSObject }
 PROCESS {
 if($_) { 
    $hashtable = $_;
    if(!$combine) {
       $output = New-Object PSObject
    }
 }
    $hashtable.GetEnumerator() | 
       ForEach-Object { Add-Member -inputObject $output `
 	  	-memberType NoteProperty -name $_.Name -value $_.Value }
    $output
 }
 #}
`

