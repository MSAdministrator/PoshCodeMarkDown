---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 1284
Published Date: 
Archived Date: 2009-08-25t01
---

# re-ip vms - 

## Description

re-ip vmware vms based on the contents of a csv

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 foreach ($entry in (import-csv "spreadsheet.csv")) {
 	$ipScript = @"
 	`$NetworkConfig = Get-WmiObject -Class Win32_NetworkAdapterConfiguration
 	`$NicAdapter = `$NetworkConfig | where {`$_.DHCPEnabled -eq "True"}
 	`$NicAdapter.EnableStatic('$entry.IP','$entry.Netmask')
 	`$NicAdapter.SetGateways('$entry.Gateway')
 	"@
 
 	Get-VM $entry.VMName | Invoke-VMScript -HostUser $entry.HU -HostPassword $entry.HP -GuestUser $gu -GuestPassword $gp $ipScript
 }
`

