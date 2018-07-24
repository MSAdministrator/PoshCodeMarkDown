---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2462
Published Date: 
Archived Date: 2011-01-22t09
---

# vmware and netapp file f - 

## Description

this file was uploaded by a powergui script editor add-on.

## Comments



## Usage



## TODO



## module

``

## Code

`#
 #
 Add-PSSnapin VMware.VimAutomation.Core
 Import-Module DataOntap
 $cred = Get-VICredentialStoreItem -Host nau-vc.naucom.com -File c:\users\aaworkman\powershell\credentials.xml
 Connect-VIServer nau-vc -User $cred.User -Password $cred.Password
 
 $esxhost = "fargo-esx2.naucom.com"
 $numberofclones = 10
 $loc = Get-Folder -Location "FargoND" | Where-Object { $_.Name -cmatch "Discovered" }
 New-Folder -Location $loc -Name "agro_desktops"
 New-ResourcePool -Location $esxhost -Name agro -CpuSharesLevel Low -MemSharesLevel Low
 $folderdsk = Get-View (Get-Datacenter -Name "FargoND" | Get-Folder -Name "agro_desktops").ID
 $pool = Get-View (Get-VMHost -name $esxhost -Location FargoND | Get-ResourcePool -Name "agro").ID
 Connect-NaController -Name fargo-nas1.naucom.com
 cd vmstore:\FargoND\agrovms\agro-gld-win7
 $vmfiles = ls -Name
 $i=1
 cd vmstore:\FargoND\agrovms\
 
 do {
 $vmname = "agro-win7-"
 $vmname += "$i"
 mkdir $vmname
 foreach($vmfile in $vmfiles){
 Start-NaClone -SourcePath /vol/agrovm_integration/agro-gld-win7/${vmfile} -DestinationPath /vol/agrovm_integration/$vmname/${vmfile}
 }
 $i++
 }
 while ($i -le $numberofclones)
 $clones = Get-NaClone
 while ($clones -ne $null) {sleep -Seconds 5; $clones = Get-NaClone}
 
 
 $esxImpl = Get-VMHost -Location FargoND | Select-Object -First 1
 $esx = Get-View $esxImpl.ID 
 $dsBrowser = Get-View $esx.DatastoreBrowser
 $ds = Get-Datastore -Name "agrovms"
 $dsView = Get-View $ds
 $datastorepath = "[" + $dsView.Summary.Name + "]"
 $searchspec = New-Object VMware.Vim.HostDatastoreBrowserSearchSpec
 $searchSpec.matchpattern = "*.vmx"
 $taskMoRef = $dsBrowser.SearchDatastoreSubFolders_Task($datastorePath, $searchSpec) 
 $task = Get-View $taskMoRef
 while ($task.Info.State -eq "running"){$task = Get-View $taskMoRef}
 $i = 1
 foreach ($file in $task.info.Result){
 $vmx = $file.FolderPath + $file.File[0].Path
 $path = $file.File[0].Path
 $guestname = [regex]"\[agrovms\] (.*)\/agro-gld-win7.vmx"
 $value = $vmx -cmatch $guestname
 $vmname = $matches[1]
 if($vmname -cmatch "win7"){ $folder = $folderdsk }
 Else { $folder = $foldersrv }
 $folder.RegisterVM_Task($vmx,$vmname,$FALSE,$pool.MoRef,$null)
 $i++
 }
 $serviceInstance = get-view ServiceInstance
 $csMgr = get-view $serviceInstance.Content.CustomizationSpecManager
 $foldername = $folderdsk
 $specname = Get-OSCustomizationSpec -Name Win7-MAK-MSDN
 $spec = $csMgr.GetCustomizationSpec($specname)
 $goldens = Get-View -Viewtype VirtualMachine -Searchroot $folderdsk.MoRef
 foreach($vm in $goldens){
 $custtask = $vm.CustomizeVM_Task($spec.Spec)
 }
`

