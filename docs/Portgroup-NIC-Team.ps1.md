---
Author: lucd
Publisher: 
Copyright: 
Email: 
Version: 1.0
Encoding: ascii
License: cc0
PoshCode ID: 1314
Published Date: 
Archived Date: 2009-09-16t16
---

# portgroup nic team - 

## Description

this script will configure the portgroup to use nic teaming with the failover depending on the duplexity of the active nic.

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 #
 #
 #
 
 $esxName = "esx41.test.local"
 $vSwitch = "vSwitch1"
 $pgname = "Net1"
 $actNIC = @("vmnic1")
 $sbyNIC = @("vmnic2")
 
 $esx = Get-VMHost $esxName | Get-View
 
 $esx.Config.Network.Vswitch | where {$_.Name -eq $vSwitch} | %{
 	if(-not $_.Spec.Policy.NicTeaming.FailureCriteria.CheckBeacon){
 		"Beacon Probing should be enabled on the vSwitch first" | Out-Host
 		exit
 	}
 }
 
 $net = Get-View $esx.configmanager.networksystem
 $portgroupspec = New-Object VMWare.Vim.HostPortGroupSpec
 $portgroupSpec.vswitchname = $vSwitch
 $portgroupspec.Name = $pgname
 $portgroupspec.policy = New-object vmware.vim.HostNetworkPolicy
 
 $portgroupspec.policy.NicTeaming = New-object vmware.vim.HostNicTeamingPolicy
 $portgroupspec.policy.NicTeaming.nicOrder = New-Object vmware.vim.HostNicOrderPolicy
 $portgroupspec.policy.NicTeaming.nicOrder.activeNic = $actNIC
 $portgroupspec.policy.NicTeaming.nicOrder.standbyNic = $sbyNIC
 
 $portgroupspec.policy.NicTeaming.failureCriteria = New-Object vmware.vim.HostNicFailureCriteria
 $portgroupspec.policy.NicTeaming.failureCriteria.checkBeacon = $true
 
 $portgroupspec.policy.NicTeaming.failureCriteria.checkDuplex = $true
 $portgroupspec.policy.NicTeaming.failureCriteria.fullDuplex = "full"
 $portgroupspec.policy.NicTeaming.failureCriteria.checkSpeed = "exact"
 $portgroupspec.policy.NicTeaming.failureCriteria.speed = 1000
 
 $portgroupspec.policy.NicTeaming.notifySwitches = $true
 
 $portgroupspec.policy.NicTeaming.policy = "failover_explicit"
 
 $portgroupspec.policy.NicTeaming.RollingOrder = $false
 
 $net.UpdatePortGroup($pgname, $PortGroupSpec)
`

