---
Author: karl prosser
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 1019
Published Date: 2009-04-13t14
Archived Date: 2012-04-24t13
---

# as function - 

## Description

as function. simple wrapper for generating the hashtables that select-object uses

## Comments



## Usage



## TODO



## function

`new-selectexpression`

## Code

`#
 #
 function new-selectexpression
 {
 if ($args.count -eq 1) { $theargs = $args[0] } else {$theargs= $args }
 if ($theargs.count -gt 1)
 {
 for($loop=0;$loop -lt ($theargs.count-1);$loop+=2)
  { 
   @{Name=$theargs[$loop];Expression=$theargs[$loop+1]} 
  }
 }
 if (!($theargs.count % 2) -eq 0) {@{Name=$input[$input.count-1];Expression= invoke-Expression "{}" } }
 }
 set-Alias as  new-selectexpression
`

