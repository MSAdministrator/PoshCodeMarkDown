---
Author: carter shanklin
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 1254
Published Date: 
Archived Date: 2009-08-10t02
---

# search cmdlet help - 

## Description

this is a simple little function to search all available cmdlets for a given keyword. similar to man -k.

## Comments



## Usage



## TODO



## function

`search-help`

## Code

`#
 #
 function Search-Help($term) {
 	Get-Command | Where { Get-Help -full -ea SilentlyContinue $_ |
 	    Out-String | Select-String $term }
 }
`

