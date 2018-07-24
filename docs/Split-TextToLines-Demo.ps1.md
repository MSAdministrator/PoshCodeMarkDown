---
Author: bernd kriszio
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 886
Published Date: 2009-02-21t10
Archived Date: 2016-07-08t10
---

# split-texttolines demo - 

## Description

demo of different ways to split a text into lines. sorry i have not yet another nice way to post code in my blog. http

## Comments



## Usage



## TODO



## function

`show-linearraystructure`

## Code

`#
 #
 function Show-LineArrayStructure ($lines)
 {
     $len = $lines.length
     "Type is:         $($lines.gettype().Name)"
     "Number of lines: $len"
     for ($i = 0; $i -lt $len; $i++)
     {
         "$($i + 1). Line: length $($lines[$i].length) >$($lines[$i])<"
     }
     '' 
     
 } 
 
 
     $text = "abc`r`nefg`r`n"
 
 
 
     $lines = $text.Split("`n")
     Show-LineArrayStructure $lines
     
 
     $lines2 = $text.Split("`r`n")
     Show-LineArrayStructure $lines2
 
     $lines3 = $([Regex]::Split($text,"`r`n" ))
     Show-LineArrayStructure $lines3
     
     $lines4 = $text -split "`n"
     Show-LineArrayStructure $lines4
 
    $lines5 = $text -split "`r`n"
     Show-LineArrayStructure $lines5
 
     $lines6 =$text.Split([string[]]"`r`n", [StringSplitOptions]::None)
     Show-LineArrayStructure $lines6
`

