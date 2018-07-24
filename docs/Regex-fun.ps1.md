---
Author: zefram
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5605
Published Date: 2015-11-20t18
Archived Date: 2015-01-31t20
---

# regex fun - 

## Description

regex match valid character string

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 $CharsString = "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz123456789!@#$%^&*()-=_+[]/\{}|:;'`",.<>?``~"
 
 a' -match "[$([regex]::escape($CharsString))]"
 > False
`

