---
Author: michael liben
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6441
Published Date: 2016-07-02t14
Archived Date: 2016-07-04t13
---

# code correction - 

## Description

correction to line 51. each octet pair should be two characters. original code omits leading zeroes in an octet pair. the format expressions should be {”{0

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 
         $escapedGuid =  "\" + ((([GUID]$guid).ToByteArray() |% {"{0:x2}" -f $_}) -join '\')
`

