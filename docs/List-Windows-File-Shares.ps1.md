---
Author: therobotdave
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 3837
Published Date: 2013-12-19t15
Archived Date: 2016-04-15t23
---

# list windows file shares - 

## Description

create excel list of file shares from remote windows server (posh one-liner)

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 Get-WmiObject Win32_Share -computerName SERVERNAME | 
 Select Name, Caption, Path | Export-csv "c:\temp\SERVERNAME.csv" -NoTypeInformation
`

