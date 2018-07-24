---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 1626
Published Date: 
Archived Date: 2010-02-26t12
---

# del. trailingblank (ise) - 

## Description

this function is intended to be uses as ise add on.

## Comments



## Usage



## TODO



## function

`delete-trailingblanks`

## Code

`#
 #
 function Delete-TrailingBlanks
 {
     $editor = $psISE.CurrentFile.Editor
     $caretLine = $editor.CaretLine
 
  
 
 
     $editor.Text = $editor.Text -replace '(?m)\s*?$', ''
 
        
     $editor.SetCaretPosition($caretLine, 1)
 }
`

