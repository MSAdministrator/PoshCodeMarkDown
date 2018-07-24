---
Author: glenn sizemore 12/19/2009
Publisher: 
Copyright: 
Email: 
Version: 1.0
Encoding: ascii
License: cc0
PoshCode ID: 743
Published Date: 2009-12-19t12
Archived Date: 2014-05-11t02
---

# update-vswitchsecurity - 

## Description

change the security setting of a vswitch.  requires v2, and the vi toolkit for windows

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 #
 Cmdlet Update-vSwitchSecurity -SupportsShouldProcess {
 	param (
 	[Parameter(position=0,Mandatory=$TRUE,HelpMessage="Name of the vSwitch to modify")]
 	[string]
 	$vSwitch,
 
 	[Parameter(position=1,Mandatory=$TRUE,ValueFromPipeline=$TRUE,HelpMessage="One or more hosts for which we want to modify the vSwitch Security")]
 	[VMware.VimAutomation.Client20.VMHostImpl[]]
 	$VMhost,
 
 	[switch]
 	$AllowPromiscuous,
 
 	[switch]
 	$MacChanges,
 
 	[switch]
 	$ForgedTransmits
 	)
 	#.Synopsis
 	#.Description
 	#.Parameter vSwitch
 	#
 	#.Parameter VMHost
 	#
 	#.Parameter AllowPromiscuous
 	#
 	#.Parameter ForgedTransmits
 	#
 	#.Parameter MacChanges
 	#
 	#.Example
 	#.Example
 	#
 	#
 	#
 	#
 	#
 	foreach ($H in $vmhost) {
 		$hostid = Get-VMHost $H | get-view
 		$networkSystem = get-view $hostid.ConfigManager.NetworkSystem
 		$networkSystem.NetworkConfig.Vswitch| ?{$_.name -match $vSwitch} | % {
 			$switchSpec = $_.spec
 			$vSwitchName = $_.name
 			if ($AllowPromiscuous) {
 				$switchSpec.Policy.Security.AllowPromiscuous = $TRUE
 				$msg = "Updating $($vSwitchName) Security settings: AllowPromiscuous=True"
 			} else {
 				$switchSpec.Policy.Security.AllowPromiscuous = $FALSE
 				$msg = "Updating $($vSwitchName) Security settings: AllowPromiscuous=False"
 			}
 			if ($MacChanges) {
 				$switchSpec.Policy.Security.MacChanges = $TRUE
 				$msg += ", MacChanges=True"
 			} else {
 				$switchSpec.Policy.Security.MacChanges = $FALSE
 				$msg += ", MacChanges=False"
 			}
 			if ($ForgedTransmits) {
 				$switchSpec.Policy.Security.ForgedTransmits = $TRUE
 				$msg += ", ForgedTransmits=True"
 			} else {
 				$switchSpec.Policy.Security.ForgedTransmits = $FALSE
 				$msg += ", ForgedTransmits=False"
 			}
 			if (($pscmdlet.ShouldProcess($H.Name, $msg))) {
 				$hostNetworkSystemView = get-view $hostid.configManager.networkSystem
 				$hostNetworkSystemView.UpdateVirtualSwitch($vSwitchName, $switchSpec)
 			}
 		}
 	}
 }
`

