---
Author: leon scheltema
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 4009
Published Date: 2013-03-12t10
Archived Date: 2016-03-25t09
---

# disable hotadd-hotplug - 

## Description

disable vmware hotadd and hotplug feature to prevent accidental removal of vm nic or disks

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 #
 ########################################################################################################
 FunctionCheckVMX
 {
 ForEach ($vmCHECK in $vmLIST)
 {
 $Error.psbase.clear()$Option = $vmCHECK.extensionData.config.extraconfig | where {$_.Key -eq "devices.hotplug"}
 If ($Option.value -match "false")
 {
 "Value already present in VMX file for " + $vmCHECK 
 }
 Else
 {
 EditVMX
 }
 }
 $Error.psbase.clear()
 
 $vm = Get-View (Get-VM $vmCHECK).ID
 $vmConfigSpec = New-Object VMware.Vim.VirtualMachineConfigSpec
 $vmConfigSpec.extraconfig += New-Object VMware.Vim.optionvalue
 $vmConfigSpec.extraconfig[0].Key="devices.hotplug"
 $vmConfigSpec.extraconfig[0].Value="false"
 $vm.ReconfigVM($vmConfigSpec)
 
 If ($Error.Count -eq 0){
 "Updated VMX File for " + $vmCHECK
 }
 Else
 {
 "Error deleting " + $vmCHECK$Error[0].exception
 }
 }########################################################################################################
 add-pssnapinVMware.VimAutomation.Core
 
 Set-PowerCLIConfiguration-InvalidCertificateAction "Ignore" -DefaultVIServerMode "single" -Confirm:$false
 
 
 $Date= get-date -uformat "%d-%m-%Y"
 $LogDir= "<Log Location>"
 $Log= $LogDir + "\EditVM-" + $Date + ".log"
 $vmLIST= Get-VM | Where-Object {$_.PowerState -eq "PoweredOff"} | Where-Object {$_.Name -like "<Client Name>"}
 
 $ErrorActionPreference= 'SilentlyContinue'
 
 CheckVMX| Out-File $Log
 
 Disconnect-VIServer-Server * -Force -Confirm:$false
 
`

