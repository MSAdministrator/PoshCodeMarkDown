---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 1512
Published Date: 
Archived Date: 2009-12-12t18
---

# multiple strings params - 

## Description

multiple string parameters

## Comments



## Usage



## TODO



## function

``

## Code

`#
 #
 
 function copySourceDestination {
 	Param (
 	 	[string]$sourceFile,
 		[string]$destinationPath
 	)
 
 
 	Copy-Item -Path:$sourceFile -Destination:$destinationPath -Force
 }
 
 
 copySourceDestination "C:\bla.txt" "\\server\share\path"
`

