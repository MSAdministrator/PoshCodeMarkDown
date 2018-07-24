---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 477
Published Date: 2008-07-23t23
Archived Date: 2011-11-03t03
---

# ellipsis - 

## Description

the infamous ellipsis function lets you pick out a single property, rather like using select -expand … except it runs in about 2/3 the time.

## Comments



## Usage



## TODO



## function

``

## Code

`#
 #
 ################################################
 ${function:...} = { process { $_.$($args[0]) } }
`

