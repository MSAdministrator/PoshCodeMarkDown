---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 1778
Published Date: 
Archived Date: 2010-04-17t07
---

# skip-object - 

## Description

skip -every 3rd, or -upto 3 at a time, or the -first 3, or the -last 3 or any combination of the above …

## Comments



## Usage



## TODO



## function

`skip-object`

## Code

`#
 #
 function Skip-Object {
 param( 
    [int]$First = 0, [int]$Last = 0, [int]$Every = 0, [int]$UpTo = 0,  
    [Parameter(Mandatory=$true,ValueFromPipeline=$true)]
    $InputObject
 )
 begin {
    if($Last) {
       $Queue = new-object System.Collections.Queue $Last
    }
    $e = $every; $UpTo++; $u = 0
 }
 process {
    $InputObject | where { --$First -lt 0 } | 
    foreach {
       if($Last) {
          $Queue.EnQueue($_)
          if($Queue.Count -gt $Last) { $Queue.DeQueue() }
       } else { $_ }
    } |
    foreach { 
       if(!$UpTo) { $_ } elseif( --$u -le 0) {  $_; $U = $UpTo }
    } |
    foreach { 
       if($every -and (--$e -le 0)) {  $e = $every  } else { $_ } 
    }
 }
 }
`

