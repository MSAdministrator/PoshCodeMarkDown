---
Author: ton siemons
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: utf-8
License: cc0
PoshCode ID: 6837
Published Date: 2017-04-11t18
Archived Date: 2017-05-26t18
---

# vmware guests subnet - 

## Description

quick and dirty script retrieves vmware host with a specific network and change the subnet of each guest. does not work if there are 2 ip addresses defined on one nic, but since there was only one of those servers i could not bother to adapt the script

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 $snapins = "vmware.vimautomation.core"
 foreach ($snapin in $snapins){if (!(Get-PSSnapin $snapin -ErrorAction SilentlyContinue)){Add-PSSnapin $snapin}}
 
 $vserver = "vmware vCenter Server"
 $vNetwork = "General_Services"
 $logfile = "d:\Scripts\log\vm.log"
 $subnet = "255.255.255.128"
 
 connect-viserver -Server $vserver
 $vms = Get-VM
 foreach ($vm in $vms){
 	$nw = $vm|Get-NetworkAdapter
 	if (($nw.networkname) -like $vNetwork){
 		$NICs = Get-WMIObject Win32_NetworkAdapterConfiguration -ComputerName $vm.name| where{$_.IPEnabled -eq �TRUE�}
 		Foreach($NIC in $NICs) {
 			try{
 				$NIC.EnableStatic($nic.ipaddress, $subnet)
 				$string = "$vm is adapted"
 				$string 
 				$string|Out-File -FilePath $logfile -Append -Encoding OEM
 			}
 			Catch{
 				$string = "$vm is not adapted"
 				$string 
 				$string|Out-File -FilePath $logfile -Append -Encoding OEM
 			}
 		}
 	}
 }
`

