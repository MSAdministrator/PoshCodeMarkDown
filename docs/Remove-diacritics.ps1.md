---
Author: grgory schiro
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: utf-8
License: cc0
PoshCode ID: 5423
Published Date: 2017-09-13t23
Archived Date: 2017-05-17t23
---

# remove diacritics - 

## Description

remove diacritics from string

## Comments



## Usage



## TODO



## function

`remove-diacritics`

## Code

`#
 #
 
 function Remove-Diacritics([string]$String)
 {
     ($String.Normalize([Text.NormalizationForm]::FormD)-replace'\p{Mn}').Normalize([Text.NormalizationForm]::FormC)
 }
`

