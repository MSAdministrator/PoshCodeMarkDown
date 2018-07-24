---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 1583
Published Date: 
Archived Date: 2010-01-22t06
---

# set vsphere cdp linkdisc - 

## Description

a script to set the cdp settings for vsphere. note that lldp is an option in vsphere, but it doesn’t work. here’s hoping for the future!

## Comments



## Usage



## TODO



## function

`set-vswitchlinkdiscovery`

## Code

`#
 #
 function set-vSwitchLinkDiscovery {
     Param (
          [Parameter(Mandatory=$true,ValueFromPipeline=$true)]$vSwitchName
         ,[Parameter(Mandatory=$true,HelpMessage="")][string]$VMBackupDestination,
 
 	$vSwitchName = "vSwitch0"
 	$linkProtocol = "cdp"
 	$linkOperation = "both"
 	$VMHost = "myhost"
 
 	$networkview = (get-vmhostnetwork -vmhost $VMHost | get-view)
 	$vSwitchSpec = ($networkView.NetworkConfig.vSwitch | Where {$_.Name -eq $vSwitchName}).Spec
 
 	$vSwitchSpec.Bridge.LinkDiscoveryProtocolConfig.protocol = $linkProtocol	$vSwitchSpec.Bridge.LinkDiscoveryProtocolConfig.operation = $linkOperation
 
 	$networkview.updateVirtualSwitch($vSwitchName,$vSwitchSpec)
 }
`

