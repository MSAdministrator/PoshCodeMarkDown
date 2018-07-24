---
Author: bernd kriszio
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 3583
Published Date: 2015-08-19t03
Archived Date: 2015-04-24t02
---

# w8 pseudostartmenu - 

## Description

this function creates or updates a combined startmenu folder, which you can attach to you toolbar (cf. http

## Comments



## Usage



## TODO



## function

`update-fakewindow8startmenu`

## Code

`#
 #
 function Update-FakeWindow8Startmenu
 {
     $StartMenu = "C:\Startmenu"
     remove-item $StartMenu -Recurse -Force
     $path1 = "$env:ProgramData\Microsoft\Windows\Start Menu\Programs"
     gci $path1 -rec | % {
         cp $_.fullname     "$StartMenu\$($_.fullname.substring($path1.Length + 1))"
     }
 
     $path2 = "$env:AppData\Microsoft\Windows\Start Menu\Programs"
     gci "$env:AppData\Microsoft\Windows\Start Menu\Programs" -Recurse | % {
         cp $_.fullname  "$StartMenu\$($_.fullname.substring($path2.Length + 1))" -ea SilentlyContinue
     }
 
 }
`

