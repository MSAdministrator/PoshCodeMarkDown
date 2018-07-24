---
Author: mow01
Publisher: 
Copyright: 
Email: 
Version: 172.30.2.112
Encoding: ascii
License: cc0
PoshCode ID: 5841
Published Date: 2016-05-02t07
Archived Date: 2016-09-07t04
---

#  - 

## Description

really validates given ip address and returns true/false.  the original script didn’t allow zeros in the ip address (eg

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 PARAM($IP=$(read-host "Enter any IP Address"))
 
 
 
 
 #[system.net.IPAddress]::tryparse($ip,[ref]$null)
 
 
 [ref]$a = $null
 [system.net.IPAddress]::tryparse($ip,$a)
`

