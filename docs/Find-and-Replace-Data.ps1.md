---
Author: danielle
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6747
Published Date: 2017-02-22t18
Archived Date: 2017-05-22t05
---

# find and replace data - 

## Description

you know unwanted data is in a string within a file, but you don’t know what it is.  you need to have it replaced with a scheduled ps1 script.

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 
 	cat ./Preamble.xml | findstr Content-ID: > ./Content-ID
 	$ID = Get-Content ./Content-ID
 	$ID = $ID.substring(13,15)
 
 	( Get-Content ./Preamble.xml ) -replace "$ID" , "$STAMP" | 
 	Set-Content ./Preamble.xml
`

