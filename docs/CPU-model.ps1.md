---
Author: greg zakharov
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5391
Published Date: 2015-08-31t09
Archived Date: 2015-01-31t07
---

# cpu model - 

## Description

lscpu is the great command but what if i just wanna see cpu model?

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 #!/bin/bash
 cat /proc/cpuinfo | grep -oP '(?<=name\s\:\s)(.*)' | uniq
`

