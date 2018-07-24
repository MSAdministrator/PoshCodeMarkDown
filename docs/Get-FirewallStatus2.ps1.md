---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 838
Published Date: 
Archived Date: 2009-02-04t18
---

# get-firewallstatus2 - 

## Description

an alternate method of querying the registry to return the firewall status (returns $true or $false). this one does not use wmi.

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 filter global:get-firewallstatus2 ([string]$computer = $env:computername)
 	{
 	if ($_) { $computer = $_ }
 
 	$reg = [Microsoft.Win32.RegistryKey]::OpenRemoteBaseKey("LocalMachine",$computer)
 
 	$firewallEnabled = $reg.OpenSubKey("System\ControlSet001\Services\SharedAccess\Parameters\FirewallPolicy\DomainProfile").GetValue("EnableFirewall")
 
 	[bool]$firewallEnabled
 	}
`

