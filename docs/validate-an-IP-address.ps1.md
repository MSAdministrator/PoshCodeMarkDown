---
Author: mow01
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6592
Published Date: 2017-10-25t01
Archived Date: 2017-03-18t05
---

# validate an ip address - 

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

