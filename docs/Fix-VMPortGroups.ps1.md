---
Author: clint jones
Publisher: 
Copyright: 
Email: 
Version: 10.6.74
Encoding: ascii
License: cc0
PoshCode ID: 4499
Published Date: 2013-09-30t19
Archived Date: 2013-10-05t04
---

# fix-vmportgroups - 

## Description

this script will help you replace old vmware portgroups with new ones and switch all vms over to these new port groups with no downtown or network traffic loss. originally written to take an environment with nonsensical, non-standardized port group naming to a proper standard. this required supporting files as are listed in the script description. there are portions that need to be filled out with relevant info to your environment, these are also outlined in the comments.

## Comments



## Usage



## TODO



## module

``

## Code

`#
 #
 #========================================================================
 #========================================================================
 
 
 $vcenteranswer = Read-Host "What vCenter server do you want to connect to (Prod, Dev, or PCI)?"
 switch ($vcenteranswer)
 	{
 	  Prod {$vcenter = "vCenter01.your.domain"}
 	  Dev  {$vcenter = "vCenter02.your.domain"}
 	  PCI  {$vcenter = "vCenter03.your.domain"}
 	}
 $cluster = Read-Host "What cluster do you want to connect to?" 		#(i.e. "Cluster1")
 $vSwitches = @()
 switch ($cluster)
 	{
 	   Cluster1 {$vSwitches = "vSwitch1","vSwitch2"}
 	   Cluster2 {$vSwitches = "vSwitch1","vSwitch2","vSwitch3"}
 	   Cluster3 {$vSwitches = "vSwitch1"}
 	}
 
 Import-Module ActiveDirectory
 Add-PSSnapin VMware.VimAutomation.Core
 Connect-VIServer -Server $vcenter -Credential (Get-Credential)
 
 New-Item -Path c:\scripts\logs\$cluster\vm_portgroup_log_failed_after.txt -Type File -Force
 New-Item -Path c:\scripts\logs\vm_portgroup_previousconfig.txt -Type File -Force
 
 foreach ($vSwitch in $vSwitches)
 	{
 	  $portgroups = Import-Csv -Path "C:\VMware_PortGroups\$cluster\$vSwitch\portgroups($cluster - $vSwitch).csv"
 	  foreach ($portgroup in $portgroups)
 		  {
 		   $hosts =  Get-Cluster $cluster | Get-VMHost
 		    foreach ($line in $hosts)
 		  	  {
 			    $vswitchName = Get-VirtualSwitch -VMHost $line -Name $vSwitch
 			    New-VirtualPortGroup -VirtualSwitch $vswitchName -Name $portgroup.Name -VLanId $portgroup.vLAN
 			  }	
 		  }
 	}
 $oldportgroups = @()
 $oldportgroups = Get-Content "C:\VMware_PortGroups\$cluster\oldgroups($cluster).txt"
 $allvms = Get-Cluster $cluster | Get-VM
 
 foreach ($vm in $allvms)
 	{
 	  $existinggroups = @()
 	  $existinggroups += Get-VirtualPortGroup -VM $vm 
 	  foreach ($existinggroup in $existinggroups)
 		{
 		  if ($oldportgroups -contains $existinggroup.Name)
 		  	{
 				    if ($cluster -eq "Cluster1")
 						{
 						 switch ($existinggroup.Name)
 							{
 						 	  	"OldGroup1" {$newgroup = "10.2.30.x 30"}
 								"OldGroup2" {$newgroup = "10.4.19.x 19"}
 								"OldGroup3" {$newgroup = "10.6.52.x 52"}
 							}
 						}
 			         if ($cluster -eq "Cluster2")
 						{
 						 switch ($existinggroup.Name)
 							{
 						 	  	"OldGroup1" {$newgroup = "10.2.42.x 42"}
 								"OldGroup2" {$newgroup = "10.4.81.x 81"}
 								"OldGroup3" {$newgroup = "10.6.74.x 74"}
 							}
 						}
 					 if ($cluster -eq "Cluster3")
 						{
 						 switch ($existinggroup.Name)
 							{
 						 	  	"OldGroup1" {$newgroup = "10.2.12.x 12"}
 								"OldGroup2" {$newgroup = "10.4.31.x 31"}
 								"OldGroup3" {$newgroup = "10.6.43.x 43"}
 							}
 						}
 					
 				$networkadapter = Get-VM $vm | Get-NetworkAdapter | Where-Object{$_.NetworkName -eq $existinggroup.Name}
 			 	Set-NetworkAdapter -NetworkAdapter $networkadapter -NetworkName $newgroup -Confirm:$false
 			}
 				Add-Content -Path c:\scripts\logs\vm_portgroup_previousconfig.txt "$vm is from the $existinggroup network."
 				Start-Sleep -Seconds 5
 				if ((Test-Connection $vm.Name -Count 2 -Quiet) -eq $false)
 				{Add-Content -Path c:\scripts\logs\$cluster\vm_portgroup_log_failed_after.txt $vm}
 		}
 	}
 
 New-Item -Path c:\scripts\logs\temp($cluster).txt -Type file
 Get-Content -Path c:\scripts\logs\$cluster\failed_connections_prior_to_script.txt | Add-Content c:\scripts\logs\temp($cluster).txt
 Get-Content -Path c:\scripts\logs\$luster\vm_portgroup_log_failed_after.txt | Add-Content c:\scripts\logs\temp($cluster).txt
 Get-Content -Path c:\scripts\logs\temp.txt | Sort-Object | Get-Unique | Out-File c:\scripts\logs\$cluster\FAILED_VM_PINGS.txt
 Remove-Item c:\scripts\logs\temp($cluster).txt -Force
`

