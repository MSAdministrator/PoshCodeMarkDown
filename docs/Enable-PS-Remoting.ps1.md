---
Author: st3v3o
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 4382
Published Date: 2016-08-12t02
Archived Date: 2016-02-14t09
---

# enable ps remoting - 

## Description

enable powershell remoting allowing access for all trusted hosts

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 echo Y | winrm quickconfig
 
 enable-psremoting -force
 
 cd wsman:
 cd localhost\client
 Set-Item TrustedHosts * -force
 restart-Service winrm
 echo "Complete"
`

