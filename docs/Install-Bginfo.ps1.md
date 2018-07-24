---
Author: chad miller
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5525
Published Date: 2015-10-19t20
Archived Date: 2015-09-19t14
---

# install-bginfo.ps1 - 

## Description

install and run bginfo at startup using registry method

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 
 if (Test-Path "C:\WINDOWS\system32\bginfo")
 { remove-item -path "C:\WINDOWS\system32\bginfo" -Recurse }
 
 copy-item \\Z001\d$\sw\bginfo -Destination C:\Windows\system32 -Recurse
 
 Set-ItemProperty -path "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Run" -name "BgInfo" -value  "C:\WINDOWS\system32\bginfo\Bginfo.exe C:\WINDOWS\system32\bginfo\bginfo.bgi /TIMER:0 /NOLICPROMPT"
`

