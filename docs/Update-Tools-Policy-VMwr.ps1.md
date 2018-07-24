---
Author: anksoswordpress
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: utf-8
License: cc0
PoshCode ID: 6152
Published Date: 2016-12-22t10
Archived Date: 2016-03-18t21
---

# update tools policy vmwr - 

## Description

this script is taking a file in .txt with the hostnames or ips and performs an update to the vm tools policy from manual to “upgrade vmware tools on power cycle”.

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 
 
 $vCname = Read-host �Enter the vCenter or ESXi host name to connect�
 Connect-viserver $vcname
 $VirtualMachines = Get-Content "VMList.txt"
 if (-not (Get-PSSnapin VMware.VimAutomation.Core -ErrorAction SilentlyContinue)) {
 	Add-PSSnapin VMware.VimAutomation.Core
 }
 foreach ($vm in $VirtualMachines ) {
 	if($_.config.Tools.ToolsUpgradePolicy -ne "UpgradeAtPowerCycle" {
 		$config = New-Object VMware.Vim.VirtualMachineConfigSpec
 		$config.ChangeVersion = $vm.ExtensionData.Config.ChangeVersion
 		$config.Tools = New-Object VMware.Vim.ToolsConfigInfo
 		$config.Tools.ToolsUpgradePolicy = "UpgradeAtPowerCycle"
 		$vm.ExtensionData.ReconfigVM($config)
 		Write-Host "Update Tools Policy on $vm completed"
 	}
 }
 Disconect-VIServer -server $vCname -Force -Confirm:$false
`

