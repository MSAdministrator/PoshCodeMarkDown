---
Author: will steele
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2795
Published Date: 2011-07-17t15
Archived Date: 2016-10-30t02
---

# time-stamp - 

## Description

this is a very simple function that returns a datetime time stamp.  i use it in scripts for noting times when actions occur like this write-host “$(time-stamp)

## Comments



## Usage



## TODO



## function

`time-stamp`

## Code

`#
 #
 function Time-Stamp
 {
     return [System.DateTime]::Now.ToString("yyyy.MM.dd hh:mm:ss");
 }
 
 New-Alias -Name ts -Value Time-Stamp;
`

