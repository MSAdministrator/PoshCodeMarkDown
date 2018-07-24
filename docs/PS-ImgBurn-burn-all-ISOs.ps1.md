---
Author: michael
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: utf-8
License: cc0
PoshCode ID: 410
Published Date: 2009-05-27t19
Archived Date: 2016-05-19t09
---

# ps+imgburn burn all isos - 

## Description

combine powershell and imbburn to burn all the iso’s in a directory. useful for burning the 7 vista iso files etc.

## Comments



## Usage



## TODO



## function

`burn-file`

## Code

`#
 #
 function burn()
 {
 dir -recurse -include *.iso -path c:\,d:\,e:\ | foreach { burn-file $_.FullName }
 }
 
 function burn-file($filename)
 {
 . "c:\Program Files\ImgBurn\ImgBurn.exe" /mode ISOWRITE /WAITFORMEDIA /start /close /DELETEIMAGE /EJECT /SRC $filename
 while ( (get-process | where{$_.ProcessName -eq "ImgBurn"}) -ne $null)
 {
 Start-Sleep â��s 10
 "waiting for $filename to complete"
 }
 
 }
 
 burn
`

