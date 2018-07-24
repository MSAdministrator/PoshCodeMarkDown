---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5159
Published Date: 
Archived Date: 2014-07-01t14
---

#  - 

## Description

copy files to dated directory.

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 $date = Get-Date -Format "yyyyMMdd"
 $source = 'C:\dir'
 $destination = "C:\someotherdir\$date\"
 
 New-Item -ItemType directory -Path $destination
 Copy-Item $source "$destination" -Recurse -Force
`

