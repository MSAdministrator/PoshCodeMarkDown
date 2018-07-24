---
Author: crazydave
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 3769
Published Date: 2013-11-16t11
Archived Date: 2017-03-18t07
---

# set-dnsserverstoopendns - 

## Description

[one-liner] sets all the local adapters to point to opendns.org’s dns servers

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 Get-WmiObject win32_networkadapterconfiguration -filter "ipenabled = 'true'" | ForEach-Object { 
     $_.SetDNSServerSearchOrder(@("208.67.222.222","208.67.220.220"));
 }
`

