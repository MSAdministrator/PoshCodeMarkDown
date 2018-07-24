---
Author: rfoust
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 836
Published Date: 2009-01-31t17
Archived Date: 2016-03-07t22
---

# get-firewallstatus - 

## Description

returns $true if the windows firewall is enabled, $false if it is disabled.

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 filter global:get-firewallstatus ([string]$computer = $env:computername)
 	{
 	if ($_) { $computer = $_ }
 
 	$HKLM = 2147483650
 
 	$reg = get-wmiobject -list -namespace root\default -computer $computer | where-object { $_.name -eq "StdRegProv" }
 	$firewallEnabled = $reg.GetDwordValue($HKLM, "System\ControlSet001\Services\SharedAccess\Parameters\FirewallPolicy\DomainProfile","EnableFirewall")
 
 	[bool]($firewallEnabled.uValue)
 	}
`

