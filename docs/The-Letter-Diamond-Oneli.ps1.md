---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 1032
Published Date: 2010-04-15t15
Archived Date: 2017-04-08t23
---

# the letter diamond oneli - 

## Description

@camurphy a slightly more elegant powershell version for his challenge

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 &{Param([char]$l)$s=66;$z=[int]$l;$o=$z-$s+ 5;$p=-1;$n=&{"$(" "*$o)A";([string[]][char[]]($s..$z))|%{$p+=2;$o--;"$(" "*$o)$_$(" "*$p)$_"}};$n;$n[$($n.Length-2)..0]}L
`

