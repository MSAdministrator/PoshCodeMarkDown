---
Author: leon scheltema
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 4192
Published Date: 2013-06-05t13
Archived Date: 2013-06-11t08
---

# vmtools update-no reboot - 

## Description

update vmtools

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 Get-VM | Get-VMGuest | Where{$_.GuestId} | Where{$_.GuestId.contains("win") -and $_.State -eq 'Running'} | Update-Tools -NoReboot
`

