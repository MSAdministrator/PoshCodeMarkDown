---
Author: bobby thing
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5920
Published Date: 2015-07-07t00
Archived Date: 2015-07-10t13
---

# convert-tochexstring - 

## Description

01,00,00,00,d0,8c,9d,df,01,15,d1,11,8c,7a,00,c0,4f,c2,97,eb,\

## Comments



## Usage



## TODO



## function

`convert-tochexstring`

## Code

`#
 #
 function Convert-ToCHexString 
 {
 	param ([String] $str) 
 	$ans = ''
 	[System.Text.Encoding]::ASCII.GetBytes($str) | % { $ans += "0x{0:X2}, " -f $_ }
 	return $ans.Trim(' ',',')
 }
`

