---
Author: himanshu
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 4163
Published Date: 2015-05-17t10
Archived Date: 2015-10-07t01
---

# custom directory in ui - 

## Description

custom directory creation using ui in power shell

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 
 $name = Read-Host 'SSIS_DUMMY?'
 $E = $name
 
 $Location = "D:\MVCApplication\"
 New-Item -Path $Location -name  $E -ItemType "directory"
`

