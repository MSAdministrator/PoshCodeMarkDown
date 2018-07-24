---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 1044
Published Date: 
Archived Date: 2009-04-22t21
---

# letterdiamondoneliner v4 - 

## Description

down to a two-statement sciptblock.

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 &{($r=,(' '*($p=[char]$args[0]-($s=65))+[char]$s)+($p..1|%{"{0,$_} {1}{0}"-f[char]++$s,('  '*$f++)}));$r[-2..-99]}J
`

