---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 1.0
Encoding: ascii
License: cc0
PoshCode ID: 5865
Published Date: 2015-05-21t03
Archived Date: 2015-05-23t02
---

# new-desktopini - 

## Description

create a desktop.ini in your powershell folder setting the icon and messing with the display name.

## Comments



## Usage



## TODO



## class

``

## Code

`#
 #
 $di = [System.IO.FileInfo]"$(split-path $Profile -Parent)\desktop.ini"
 set-content $di "[.ShellClassInfo]`r`nLocalizedResourceName=1$([char]160)WindowsPowerShell`r`nIconResource=C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe,0`r`n[ViewState]`r`nFolderType=Documents"
 $di.Attributes = $di.Attributes -bor [IO.FileAttributes]"System,Hidden" -bxor [IO.FileAttributes]"Archive"
`

