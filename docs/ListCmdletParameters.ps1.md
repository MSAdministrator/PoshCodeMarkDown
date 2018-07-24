---
Author: sean kearney
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: utf-8
License: cc0
PoshCode ID: 3188
Published Date: 2012-01-25t21
Archived Date: 2012-01-29t07
---

# listcmdletparameters - 

## Description

extract all parameters for a cmdlet from get-help, list them in a single column

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 #
 #
 #
 #
 #
 param($HelpData)
 
 ($HelpData).Syntax | SELECT-OBJECT �ExpandProperty SyntaxItem | SELECT-OBJECT �ExpandProperty parameter | SELECT-OBJECT name
`

