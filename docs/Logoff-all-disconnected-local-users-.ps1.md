---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6228
Published Date: 
Archived Date: 2016-08-26t03
---

#  - 

## Description

logoff all disconnected local users, inspired by this

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 $UserSessions = query.exe session | Select-Object -Skip 1
 foreach ($SessionString in $UserSessions) {
     $Session = $SessionString.Split(" ",[System.StringSplitOptions]::RemoveEmptyEntries) 
     if (($Session[2] -eq "Disc") -and ($Session[0] -ne "services"))   {
         logoff.exe $Session[1] /V
     }
 }
`

