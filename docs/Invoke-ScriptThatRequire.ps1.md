---
Author: lee holmes
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2187
Published Date: 2011-09-09t21
Archived Date: 2016-03-19t02
---

# invoke-scriptthatrequire - 

## Description

from windows powershell cookbook (o’reilly) by lee holmes

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 ###########################################################################
 ##
 ##
 ##
 ###########################################################################
 
 <#
 
 .SYNOPSIS
 
 Demonstrates a technique to relaunch a script that requires STA mode.
 This is useful only for simple parameter definitions that can be
 specified positionally.
 
 #>
 
 param(
     $Parameter1,
     $Parameter2
 )
 
 Set-StrictMode -Version Latest
 
 "Current threading mode: " + $host.Runspace.ApartmentState
 "Parameter1 is: $parameter1"
 "Parameter2 is: $parameter2"
 
 if($host.Runspace.ApartmentState -ne "STA")
 {
     "Relaunching"
     $file = $myInvocation.MyCommand.Path
     powershell -NoProfile -Sta -File $file $parameter1 $parameter2
     return
 }
 
 "After relaunch - current threading mode: " + $host.Runspace.ApartmentState
`

