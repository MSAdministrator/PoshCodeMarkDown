---
Author: olivia wild
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6246
Published Date: 2016-03-05t17
Archived Date: 2016-10-18t13
---

# antivirus path - 

## Description

this script returns a path where antivirus has been installed. it doesn’t use wmi (thanks a lot to greg zakharov for this trick).

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 Get-ChildItem Registry::HKCR\CLSID | ForEach-Object {
   $x = (Get-ItemProperty 'Registry::HKCR\Component Categories\*' |
   Where-Object {$_ -match 'antivirus'}).PSChildName
 }{
   if ((Get-ChildItem "$($_.PSPath)\Implemented Categories" -ea 0).PSChildName -eq $x) {
     Split-Path (Get-ItemProperty "$($_.PSPath)\InprocServer32").'(default)'
     break
   }
 }
`

