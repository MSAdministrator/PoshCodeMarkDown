---
Author: carter shanklin
Publisher: 
Copyright: 
Email: 
Version: 10.91
Encoding: ascii
License: cc0
PoshCode ID: 6455
Published Date: 2016-07-25t21
Archived Date: 2016-07-29t20
---

# get-observediprange - 

## Description

get observed ip address ranges and vlan ids from an esx host’s physical adapter. sample use at the bottom.

## Comments



## Usage



## TODO



## function

`get-observediprange`

## Code

`#
 #
 function Get-ObservedIPRange {
 	param(
 		[Parameter(Mandatory=$true,ValueFromPipeline=$true,HelpMessage="Physical NIC from Get-VMHostNetworkAdapter")]
 		[VMware.VimAutomation.Client20.Host.NIC.PhysicalNicImpl]
 		$Nic
 	)
 
 	process {
 		$hostView = Get-VMHost -Id $Nic.VMHostId | Get-View -Property ConfigManager
 		$ns = Get-View $hostView.ConfigManager.NetworkSystem
 		$hints = $ns.QueryNetworkHint($Nic.Name)
 
 		foreach ($hint in $hints) {
 			foreach ($subnet in $hint.subnet) {
 				$observed = New-Object -TypeName PSObject
 				$observed | Add-Member -MemberType NoteProperty -Name Device -Value $Nic.Name
 				$observed | Add-Member -MemberType NoteProperty -Name VMHostId -Value $Nic.VMHostId
 				$observed | Add-Member -MemberType NoteProperty -Name IPSubnet -Value $subnet.IPSubnet
 				$observed | Add-Member -MemberType NoteProperty -Name VlanId -Value $subnet.VlanId
 				Write-Output $observed
 			}
 		}
 	}
 }
 
 
 #------                 --------               --------                               ------
`

