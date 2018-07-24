---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2516
Published Date: 2011-02-23t07
Archived Date: 2015-04-20t01
---

# join-string - 

## Description

joins array elements together using a specific string separator

## Comments



## Usage



## TODO



## function

`join-string`

## Code

`#
 #
 #.Synopsis
 #.Example
 #.Example
 #
 
 function Join-String { 
    param    ( [string]$separator, [string]$append, [string]$prepend, [string]$prefix, [string]$postfix, [switch]$unique )
    begin    { [string[]]$items =  @($prepend.split($separator)) }
    process  { $items += $_ }
    end      { 
       $ofs = $separator; 
       $items += @($append.split($separator)); 
       if($unique) {
          $items  = $items | Select -Unique
       }
       return "$prefix$($items -ne '')$postfix"
    }
 }
`

