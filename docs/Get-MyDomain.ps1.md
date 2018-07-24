---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 731
Published Date: 
Archived Date: 2008-12-18t09
---

# get-mydomain - 

## Description

get-mydomain retrieves the current ip of the user (or, the first if there are multiple active cards) then performs a dns lookup to retrieve the domain. if it is unable to reverse it, it displays unknown

## Comments



## Usage



## TODO



## function

`get-mydomain`

## Code

`#
 #
 Function Get-MyDomain() {
 $IP = ((Get-WmiObject -Class Win32_NetworkAdapterConfiguration -Filter IPEnabled=TRUE -ComputerName . | Select-Object -Property IPAddress -First 1).IPAddress[0])
   trap {
     return "Unknown:$($IP)"
 	}
 return [System.Net.DNS]::GetHostByAddress($IP).HostName
 }
`

