---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 1619
Published Date: 
Archived Date: 2010-02-03t15
---

# write-isefile - 

## Description

if you are using ise put this file anywhere into your path and functions depending on it can use it.

## Comments



## Usage



## TODO



## function

`write-isefile`

## Code

`#
 #
 function Write-IseFile($file, $msg)
 {
     $Editor = $file.Editor
     $Editor.SetCaretPosition($Editor.LineCount, 1)
     $Editor.InsertText(($msg  + "`r`n"))
 }
`

