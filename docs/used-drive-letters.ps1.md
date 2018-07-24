---
Author: greg zakharov
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5110
Published Date: 2014-04-22t18
Archived Date: 2014-04-26t17
---

# used drive letters - 

## Description

fix

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 [Char[]](65..90) | ? {cmd /c 2`>nul @($_ + ':') `&`& echo $_}
`

