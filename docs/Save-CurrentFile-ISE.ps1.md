---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 1318
Published Date: 
Archived Date: 2009-09-16t23
---

# save-currentfile (ise) - 

## Description

why to use a fileselectionbox to save your fresh files from ise, don’t you know your file system? ok perhaps an encoding parameter would be fine, but please don’t default it to ascii.

## Comments



## Usage



## TODO



## function

`save-currentfile`

## Code

`#
 #
 function Save-CurrentFile ($path)
 {
     $psISE.CurrentFile.SaveAs($path)
     $psISE.CurrentFile.Save([Text.Encoding]::default)
 
 }
 
`

