---
Author: greg zakharov
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 4827
Published Date: 2014-01-22t11
Archived Date: 2014-02-05t10
---

# bash - 

## Description

prints full path for each file in a directory

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 #!/bin/bash
 
 for file in $(ls -a "$@"); do
 	echo -n $(pwd)
 	[[ $(pwd) != "/" ]] && echo -n /
 	echo $file
 done
`

