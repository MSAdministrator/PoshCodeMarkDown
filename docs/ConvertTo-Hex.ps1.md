---
Author: s-1-5-21-2025429
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6847
Published Date: 2017-04-18t09
Archived Date: 2017-04-21t20
---

# convertto-hex - 

## Description

this scconvertto-hex script will convert a security identifier (sid) in string format to its hexadecimal equivalent. e.g.

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 param ( [string]$SidString )
 
 $sid = New-Object system.Security.Principal.SecurityIdentifier $sidstring
 
 $sidBytes = New-Object byte[] $sid.BinaryLength
 
 $sid.GetBinaryForm( $sidBytes, 0 )
 
 $hexArr = $sidBytes | ForEach-Object { $_.ToString("X2") }
 
 $hexArr -join ''
`

