---
Author: piere woehl
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5647
Published Date: 2016-12-15t19
Archived Date: 2016-09-07t04
---

# get-foldersize - 

## Description

a little script for $profile file to add support for get-foldersize.

## Comments



## Usage



## TODO



## function

`get-foldersize`

## Code

`#
 #
 function Get-FolderSize {
 $location = $args[0]
 Write-Host "Directory to Scan:"$location
 $value = "{0:N2}" -f ((Get-ChildItem -recurse $location | Measure-Object -property length -sum).Sum / 1MB)
 Write-Host "Used Storage for Directory:"$value" MB"
 }
`

