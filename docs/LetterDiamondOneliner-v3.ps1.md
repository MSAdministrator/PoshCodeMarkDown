---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 1040
Published Date: 
Archived Date: 2009-04-23t15
---

# letterdiamondoneliner v3 - 

## Description

@karl

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 &{param([char]$c)[int]$s=65;$p=$c-$s;$r=,(' '*$p+[char]$s);$r+=@(do{"{0,$p} {1}{0}"-f([char]++$s),('  '*$f++)}until(!--$p));$r;$r[-2..-99]}Z
 
 &{param([char]$c)$p=$c-($s=65);$r=,(' '*$p+[char]$s);do{$r+="{0,$p} {1}{0}"-f([char]++$s),('  '*$f++)}until(!--$p);$r;$r[-2..-99]}J
 
 &{$p=[char]$args[0]-($s=65);$r=,(' '*$p+[char]$s);do{$r+="{0,$p} {1}{0}"-f([char]++$s),('  '*$f++)}until(!--$p);$r;$r[-2..-99]}J
`

