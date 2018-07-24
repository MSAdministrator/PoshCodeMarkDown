---
Author: leon scheltema
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5664
Published Date: 2015-01-07t10
Archived Date: 2015-01-15t20
---

# step02a-export_vm-to-fol - 

## Description

export vcenter folder structure incl vm relations

## Comments



## Usage



## TODO



## function

`copy-vcfolderstructure`

## Code

`#
 #
 
 $DefaultVIServers
 $OldVC = "Old vCenter"
 $NewVC = "New vCenter"
 
 Connect-VIServer "$OldVC"
 Connect-VIServer "$NewVC"
 
 
 function Copy-VCFolderStructure {
 <#
     .SYNOPSIS
         Copy-VCFolderStructure copies folder and its structure from one VC to another..
  
     .DESCRIPTION
         Copy-VCFolderStructure can be handy when doing migrations of clusters/hosts between
         Virtual Center servers. It takes folder structure from 'old' VC and it recreates it on 'new'
         VC. While doing this it will also output virtualmachine name and folderid. Why would you
         want to have it ? Let's say that you have a cluster on old virtual center server 
         oldvc.local.lab
         DC1\Cluster1\folder1
         DC1\Cluster1\folderN\subfolderN
         Copy-VCFolderStructure will copy entire folder structure to 'new' VC, and while doing this
         it will output to screen VMs that resides in those structures. VM name that will be shown on
         screen will show also folderid, this ID is the folderid on new VC.  After you have migrated 
         your hosts from old cluster in old VC to new cluster in new VC, and folder structure is there,
         you can use move-vm cmdlet with -Location parameter. As location you would have give the
         folder object that corresponds to vm that is being moved. Property Name is the name of VM
         that was discovered in that folder and Folder is the folderid in which the vm should be moved
         into. This folderid has to first changed to folder, for example :
         $folderobj=get-view -id $folder|Get-VIObjectByVIView
         We can then use $folderobj as parameter to move-vm Location parameter
  
     .PARAMETER  OldFolder
         This should be the extensiondata of folder that you want to copy to new VC.
         $folderToRecreate=Get-Folder -Server oldVC.lab.local -Name teststruct
         Have in mind that this should be an single folder and not an array.
          
  
     .PARAMETER  ParentOfNewFolder
         When invoking the function this is the root folder where you want to attach the copied folder.
         Let's say you are copying folder from \DatacenterA\FolderX\myfolder
         If you will have the same structure on the new VC, you would have set ParentOfNew folder
         to FolderX. Still it's not a problem if you have a new structure on new VC. Let's say that on
         new VC you have folder: \DatacenterZ\NewStructure\FolderZ and you want to copy entire
         'myfolder' beneath the FolderZ. In that case, first create a variable that has desired folder
         $anchor=get-folder 'FolderZ' -Server newVC 
         Make sure that $anchor variable will have only 1 element.
          
     .PARAMETER  NewVC
         This parameter describes virtual center to which we are copying the folder structure.
         Copy-VCFolderStructure works only when you are connected to both old and new vc at the
         same time. You need to set your configuration of PowerCLI to handle multiple connections.
         Set-PowerCLIConfiguration -DefaultVIServerMode 'Multiple'
         You can check if you are connected to both servers using $global:DefaultVIServers variable
  
     .PARAMETER  OldVC
         This parameter describes virtual center from which we are copying the folder structure.
         Copy-VCFolderStructure works only when you are connected to both old and new vc at the
         same time. You need to set your configuration of PowerCLI to handle multiple connections.
         Set-PowerCLIConfiguration -DefaultVIServerMode 'Multiple'
         You can check if you are connected to both servers using $global:DefaultVIServers variable
          
          
  
     .EXAMPLE
         PS C:\> Set-PowerCLIConfiguration -DefaultVIServerMode 'multiple'
         PS C:\> $DefaultVIServers 
         Ensure that you are connected to both VC servers
         Establish variables:
         This will be the folder that we will be copying from old VC
         $folderToRecreate=Get-Folder -Server $OldVC -Name 'teststruct'
         This will be the folder to which we will be copying the folder structure
         $anchor=get-folder 'IWantToPutMyStructureHere' -Server $NewVC
         $OldVC='myoldvc.lab.local'
         $NewVC='mynewvc.lab.local'
         Copy-VCFolderStructure -OldFolder $folderToRecreate.exensiondata -NewVC $NewVC 
         -OldVC $OldVC -ParentOfNewFolder $anchor
         $OldFolder expects to get exensiondata object from the folder, if you will not provide it, function will
         block it.
  
     .EXAMPLE
         If you are planning to move vms after hosts/vm/folders were migrated to new VC, you might use it in this way.
         By default Copy-VCFolderStructure will output also vms and their folder ids in which they should reside on new
         VC. You can grab them like this:
         $vmlist=Copy-VCFolderStructure -OldFolder $folderToRecreate.exensiondata -NewVC $NewVC -OldVC $OldVC -ParentOfNewFolder $anchor
         You can now export $vmlist to csv
         $vmlist |export-csv -Path 'c:\migratedvms.csv' -NoTypeInformation
         And once all virtual machines are in new virtual center, you can import this list and do move-vm operation on those
         vms. Each vm has name and folder properties. Folder is a folderid value, which has to be converted to Folder object.
         move-vm -vm $vmlist[0].name -Location (get-view -id $vmlist[0].folder -Server $newVC|get-viobjectbyviview) -Server $newVC
         This would move vm that was residing in previously on old VC in migrated folder to its equivalent on new VC.
  
     .NOTES
         NAME:  Copy-VCFolderStructure
          
         AUTHOR: Grzegorz Kulikowski
          
          
         THANKS: Huge thanks go to Robert van den Nieuwendijk for helping me out with the recursion in this function.
  
     .LINK
  
 http://psvmware.wordpress.com
  
 #>
  
    param(
    [parameter(Mandatory = $true)]
    [ValidateNotNullOrEmpty()]
    [VMware.Vim.Folder]$OldFolder,
    [parameter(Mandatory = $true)]
    [ValidateNotNullOrEmpty()]
    [VMware.VimAutomation.ViCore.Impl.V1.Inventory.FolderImpl]$ParentOfNewFolder,
    [parameter(Mandatory = $true)]
    [ValidateNotNullOrEmpty()]
    [string]$NewVC,
    [parameter(Mandatory = $true)]
    [ValidateNotNullOrEmpty()]
    [string]$OldVC
    )
   $NewFolder = New-Folder -Location $ParentOfNewFolder -Name $OldFolder.Name -Server $NewVC
   Get-VM -NoRecursion -Location ($OldFolder|Get-VIObjectByVIView) -Server $OldVC|Select-Object Name, @{N='Folder';E={$NewFolder.id}}
   foreach ($childfolder in $OldFolder.ChildEntity|Where-Object {$_.type -eq 'Folder'})
                   {
                    Copy-VCFolderStructure -OldFolder (Get-View -Id $ChildFolder -Server $OldVC) -ParentOfNewFolder $NewFolder -NewVC $NewVC -OldVC $OldVC
                   }
 }
 
 
 $folderToRecreate=Get-Folder -Server $OldVC
 $anchor=get-folder vm -Server $NewVC
 $vmlist=Copy-VCFolderStructure -OldFolder $folderToRecreate.extensiondata -NewVC $NewVC -OldVC $OldVC -ParentOfNewFolder $anchor 
 $vmlist |export-csv -Path "migratedvms.csv" -NoTypeInformation
 
 Disconnect-VIServer -server "*" -Force -Confirm:$false
`

