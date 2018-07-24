---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6010
Published Date: 
Archived Date: 2016-05-17t13
---

#  - 

## Description

so many holla dollas

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 $30DaysFiles = Get-ChildItem $dataLocation | Where-object {
     ([datetime]::ParseExact($_.Name.Substring(4,6),"yyMMdd",$null) -gt (Get-Date).AddDays(-31)) -and `
     ([datetime]::ParseExact($_.Name.Substring(4,6),"yyMMdd",$null) -lt (Get-Date)) -and`
     
 
     (($db | Where-Object {$_.Date -eq [datetime]::ParseExact($_.Name.Substring(4,6),"yyMMdd",$null).ToString("MM/dd/yyyy")}) -eq $null)
     }
`

