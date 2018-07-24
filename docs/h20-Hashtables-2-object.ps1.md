---
Author: nqpuddji
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2697
Published Date: 2012-05-27t11
Archived Date: 2016-05-19t15
---

# h20 -hashtables 2 object - 

## Description

hashtable to object function.

## Comments



## Usage



## TODO



## function

``

## Code

`#
 #
 function h20([scriptblock]$sb )
 {
  begin {}
  process{ if ($sb -ne $null)
                 {
                   $ht = &$sb;
                   if ($ht -is [hashtable])
                     {
                         New-Object PSObject -Property $ht}
                     }
                   if ($ht -is [object[]])
                     {
                     $ht | % { New-Object PSObject -Property $_ }
                     }  
                 }
             
  end{}
 }
`

