---
Author: neiljmorrow
Publisher: 
Copyright: 
Email: neiljmorrow@gmail.com
Version: 1.0
Encoding: ascii
License: cc0
PoshCode ID: 837
Published Date: 
Archived Date: 2009-02-04t17
---

# convert-cbz2cbr - convert-cbz2cbr.ps1

## Description

converts cbz files to cbr

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 ###########################################################################
 #
 #
 #
 #
 #
 #
 #
 ###########################################################################
 
 set-alias cbrzip "c:\program files\7-zip\7z.exe"
 
 dir "*.cbz" | foreach-object{
 
 	copy-item $_.name ($_.basename + ".zip")
 
 	cbrzip -oTemp x ($_.basename + ".zip")
 	
 	remove-item ($_.basename + ".zip")
 	
 	cbrzip a ($_.basename + ".rar") "temp"
 
 	remove-item -r "temp"
 
 	rename-item ($_.basename + ".rar") ($_.basename + ".cbr")
 }
 
 remove-item alias:cbrzip
`

