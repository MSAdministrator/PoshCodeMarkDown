---
Author: greg zakharov
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 4561
Published Date: 2013-10-26t16
Archived Date: 2013-11-04t14
---

# directories tree - 

## Description

if ‘tree’ pocket has not been installed that you can use next script but note that it gets only directories.

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 #!/bin/bash
 
 if [ -z "$1" ]; then loc=$(pwd); else loc=$1; fi
 ls -aR $loc | grep ':$' | sed -e 's/:$//;s/[^-][^\/]*\//--/g;s/^/ /;s/-/|/'
`

