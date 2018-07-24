---
Author: clint jones
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 3445
Published Date: 2012-06-02t08
Archived Date: 2012-06-07t19
---

# bulk storage vmotion - 

## Description

bulk storage vmotion by cluster (staggered).

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 #========================================================================
 #========================================================================
 
 Add-PSSnapin VMware.VimAutomation.Core
 $creds = Get-Credential
 $viserver = Read-Host "vCenter Server:"
 Connect-VIServer -Server $viserver -Credential $creds
 
 $cluster = Read-Host "What cluster do you want to migrate:"
 $vms = Get-Cluster -Name $cluster | Get-VM
 
 foreach($vm in $vms)
 {
 	$currentdata = Get-VM -Name $vm.Name | Get-Datastore
 	$currentdata = $currentdata.Name
 	if ($currentdata.EndsWith("a") -eq "True")
 	{$destdata = $destdata1}
 	else
 	{$destdata = $destdata2}
 	$task = Get-VM -Name $vm.Name | Move-VM -Datastore (Get-Datastore -Name $destdata)
 	Wait-Task -Task $task
 }
`

