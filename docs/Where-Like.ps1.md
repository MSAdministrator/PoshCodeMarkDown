---
Author: sibroller
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 4283
Published Date: 2013-07-02t08
Archived Date: 2013-07-09t04
---

# where-like - 

## Description

where-like function works as a pipeline console filter.

## Comments



## Usage



## TODO



## function

`where-like`

## Code

`#
 #
 function Where-Like {
 	Param($member, $string)
 	process { $input | where {$_.$member -like $string} } 
 }
`

