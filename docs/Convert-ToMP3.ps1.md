---
Author: crazydave
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 1561
Published Date: 2010-12-27t19
Archived Date: 2017-03-18t07
---

# convert-tomp3 - 

## Description

this script uses vlc to convert an audio file to mp3 format. it makes the assumption that you have an alias “vlc” that points to the vlc executable.

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 param([String] $inputPath, [String] $wildcard, [String] $outputPath = $inputPath)
 
 gci -path $inputPath\$wildcard | % {  
   $outputFile = Join-Path $outputPath ($_.Name.Replace($_.Extension, '.mp3'))  
   Get-Process vlc | % { $_.WaitForExit() }
 }
`

