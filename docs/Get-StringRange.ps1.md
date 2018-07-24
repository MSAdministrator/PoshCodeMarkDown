---
Author: halr9000
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 1216
Published Date: 
Archived Date: 2009-10-25t10
---

# get-stringrange - 

## Description

works like the integer range operator “..”, but for characters.

## Comments



## Usage



## TODO



## function

`get-charrange`

## Code

`#
 #
 function Get-CharRange ( [char]$Start, [char]$End ) {
 	[char[]]($Start..$End)
 }
 
 function Get-LetterRange ( [char]$Start, [char]$End, [string]$charset = "BasicLatin" ) {
  [char[]]($Start..$End) | Where { $_ -match "(?=\p{Is$charset})\p{L}" }
 }
`

