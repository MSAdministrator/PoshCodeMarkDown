---
Author: oisin grehan
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 3827
Published Date: 2013-12-18t07
Archived Date: 2017-05-13t16
---

# force 64 bit for script - 

## Description

place this at the head of a ps1 script to ensure that if it’s run in a 32 bit shell (like the nuget package manage console for example) that it will be relaunched in a 64 bit powershell process, and will it will also pass along any arguments to the child process (please use primitives like string, bool etc for arguments as they are being marshaled to another win32 process – live objects will not work.)

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 if ($pshome -like "*syswow64*") {
 	
 	write-warning "Restarting script under 64 bit powershell"
 
 	& (join-path ($pshome -replace "syswow64", "sysnative") powershell.exe) -file `
 		(join-path $psscriptroot $myinvocation.mycommand) @args
 
 	exit
 }
 
 
 write-warning "hello from $pshome"
 write-warning "My original arguments $args"
`

