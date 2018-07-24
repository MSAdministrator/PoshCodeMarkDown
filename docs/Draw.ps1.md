---
Author: nathan estell
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6383
Published Date: 2016-06-14t18
Archived Date: 2016-06-17t09
---

# draw - 

## Description

turns a here-string with characters, spaces, and new lines into an image with blocks of color, spaces, and new lines.

## Comments



## Usage



## TODO



## function

``

## Code

`#
 #
 function Draw
 {
 param
 (
 $text=@"
 q   
  q  
   q 
    q
 "@
 )
 $CharArray=$text.tochararray()
 foreach ($character in $CharArray)
 {
 if ($character -match "\S" )
 {
 write-host " " -BackgroundColor "Green" -NoNewLine
 #"Character"
 }
 if ($character -match "[^\S\r\n]")
 {
 write-host " " -BackgroundColor "Black" -NoNewLine
 #"Space"
 }
 if ($character -match "\n" )
 {
 write-host "" 
 #"New Line"
 }
 }
 }
`

