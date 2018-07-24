---
Author: greg zakharov
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 4829
Published Date: 2014-01-22t11
Archived Date: 2014-01-29t05
---

# bash - 

## Description

shows results of a command in different command shell window

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 #!/bin/bash
 
 gnome-terminal -x bash -c "ls -a;echo Press any key to continue...;read"
 
 xfce4-terminal -x bash -c "ls -a;echo Press any key to continue...;read"
`

