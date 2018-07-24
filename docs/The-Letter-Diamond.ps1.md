---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 1029
Published Date: 2010-04-15t07
Archived Date: 2017-04-10t19
---

# the letter diamond - 

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
 Param([char]$letter = "E", [int]$padding=5)
 $start = [int][char]"B"
 $end = [int]$letter
 
 $outerpadding = ($end - $start) + $padding
 $innerpadding = -1
 
 $lines = &{ 
    "$(" " * $outerpadding)A"
    foreach($char in ([string[]][char[]]($start..$end))) { 
       $innerpadding += 2; $outerpadding--
       "$(" " * $outerpadding)$char$(" " * $innerpadding)$char"
    }
 }
 
 $lines
 $lines[$($lines.Length-2)..0]
`

