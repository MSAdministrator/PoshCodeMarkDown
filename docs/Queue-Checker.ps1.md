---
Author: littlegun
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 3640
Published Date: 2012-09-14t11
Archived Date: 2012-09-21t03
---

# queue checker - 

## Description

checks all exchange queues in an organization

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 foreach
 ($ExchangServer in (Get-ExchangeServer | Where { $_.isHubTransportServer -eq $True})) 
 {Get-queue -Server $ExchangeServer}
`

