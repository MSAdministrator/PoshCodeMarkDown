---
Author: greg zakharov
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5063
Published Date: 2014-04-08t11
Archived Date: 2014-04-13t13
---

# ip - 

## Description



## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 <#
 #
 #
 #>
 gp HKLM:\SYSTEM\CurrentControlSet\Services\Tcpip\Parameters\Interfaces\* | % {if (($ip = $_.DhcpIPAddress) -ne '0.0.0.0') {$ip}}
`

