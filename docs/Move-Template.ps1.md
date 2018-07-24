---
Author: glnsize
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 1600
Published Date: 
Archived Date: 2010-01-27t03
---

# move-template - 

## Description

v2 advanced function based off http

## Comments



## Usage



## TODO



## function

`move-template`

## Code

`#
 #
 Function Move-Template{
     <#
     .Synopsis
         Move a VM template
     .Description
         Move a VM template either to a new host via vmotion or to a new datastore via svmotion
     .Parameter Name
         Name of the VM to be migrated
     .Parameter VIObject
         Template to be migrated
     .Parameter VMHost
         VMHost to vmotion the template to
     .Parameter Datastore
         Datastore to svmotion template to
     .Example
         Move-Template -Name W2k8R201 -VMHost (Get-VMHost ESX1)
         
         VMotion W2k8R201 template to the ESX1 host.
     .Example
         Move-Template -Name W2k8R201 -Datastore (Get-Datastore NFS2)
         
         sVMotion W2k8R201 template to the NFS2 Datastore.
     .Example
         Get-Template | Move-Template -Datastore (Get-Datastore NFS2)
         
         sVMotion all templates onto the NFS2 Datastore
     .Example
         Get-Template CentOS_5_2_x64 | Move-Template -Datastore (Get-Datastore NFS2) -VMHost (Get-VMHost ESX03)
         
         sVMotion the CentOS_5_2_x64 templates onto the NFS2 Datastore, and then vmotion said template onto ESX03
     .link
         Get-Template
         New-Template
         Remove-Template
         Set-Template
     #> 
     [CmdletBinding(SupportsShouldProcess=$true,SupportsTransactions=$False,ConfirmImpact="low",DefaultParameterSetName="")]
     param( 
         [Parameter(Mandatory=$true,Position=0,HelpMessage="Name of template to move", ParameterSetName="ByName")]
         [STRING]
         $Name,
         
         [Parameter(Mandatory=$true,ValueFromPipeline=$true, HelpMessage="Managed object of the template to be migrated", ParameterSetName="ByObject")]
         [VMware.VimAutomation.Client20.TemplateImpl[]]
         $Template,
         
         [Parameter(Mandatory=$false,ValueFromPipeline=$true, HelpMessage="VMHost to vmotion the template to", ParameterSetName="ByObject")]
         [Parameter(Mandatory=$false,ValueFromPipeline=$true, HelpMessage="VMHost to vmotion the template to", ParameterSetName="ByName")]
         [VMware.VimAutomation.Client20.VMHostImpl]
         $VMHost,
         
         [Parameter(Mandatory=$false,ValueFromPipeline=$true, HelpMessage="Datastore to svmotion the template to", ParameterSetName="ByObject")]
         [Parameter(Mandatory=$false,ValueFromPipeline=$true, HelpMessage="Datastore to svmotion the template to", ParameterSetName="ByName")]
         [VMware.VimAutomation.Client20.DatastoreImpl]
         $Datastore
     )
     Process{
         switch ($pscmdlet.parametersetname) {
             "ByName"
             {
                 if ($pscmdlet.ShouldProcess($Name,"vmotion Template to $($VMHost.name) and svmotion to $($Datastore.name)")){
                     $template = Get-Template -Name $Name -verbose:$False
                     $vm = Set-Template -Template $template -ToVM -verbose:$False -confirm:$False  
                     
                     switch ($VM) {
                         {$Datastore} {Move-VM -VM $VM -Datastore $Datastore -verbose:$False -Confirm:$false}
                         {$VMHost} {Move-VM -VM $VM -Destination $VMHost -verbose:$False -Confirm:$false}
                     }
                     (Get-View -VIObject $VM -verbose:$False).MarkAsTemplate()
                 }
             }
             "ByObject"
             {
                 Foreach ($tmp in $Template) {
                     if ($pscmdlet.ShouldProcess($tmp.Name,"vmotion Template to $($VMHost.name) and svmotion to $($Datastore.name)")){
                         $vm = Set-Template -Template $tmp -ToVM -verbose:$False -confirm:$False  
                         switch ($VM) {
                             {$Datastore} {Move-VM -VM $VM -Datastore $Datastore -verbose:$False -Confirm:$false}
                             {$VMHost} {Move-VM -VM $VM -Destination $VMHost -verbose:$False -Confirm:$false}
                         }
                         (Get-View -VIObject $VM -verbose:$False).MarkAsTemplate()
                     }
                 }
             }
         }
     }
 }
`

