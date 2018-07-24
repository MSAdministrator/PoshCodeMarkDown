---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 4118
Published Date: 
Archived Date: 2013-04-23t23
---

#  - 

## Description

add vm to drs group function

## Comments



## Usage



## TODO



## function

`get-drsgroup`

## Code

`#
 #
 Function Get-DrsGroup {
 <#
 .SYNOPSIS
 Retrieves DRS groups from a cluster.
 
 .DESCRIPTION
 Retrieves DRS groups from a cluster.
 
 .PARAMETER Cluster
 Specify the cluster for which you want to retrieve the DRS groups
 
 .PARAMETER Name
 Specify the name of the DRS group you want to retrieve.
 
 .EXAMPLE
 Get-DrsGroup -Cluster $Cluster -Name "VMs DRS Group"
 Retrieves the DRS group "Vms DRS Group" from cluster $Cluster.
 
 .EXAMPLE
 Get-Cluster | Get-DrsGroup
 Retrieves all the DRS groups for all clusters.
 
 .INPUTS
 ClusterImpl
 
 .OUTPUTS
 ClusterVmGroup
 ClusterHostGroup
 
 .COMPONENT
 VMware vSphere PowerCLI
 #>
 
   param([parameter(Mandatory=$true, ValueFromPipeline=$true)]$Cluster,
         [string] $Name="*")
 
   process {
     $Cluster = Get-Cluster -Name $Cluster
     if($Cluster) {
       $Cluster.ExtensionData.ConfigurationEx.Group | `
       Where-Object {$_.Name -like $Name}
     }
   }
 }
   
 Function Add-VMToDrsGroup {
 <#
 .SYNOPSIS
 Adds a virtual machine to a cluster VM DRS group.
 
 .DESCRIPTION
 Adds a virtual machine to a cluster VM DRS group.
 
 .PARAMETER Cluster
 Specify the cluster for which you want to retrieve the DRS groups
 
 .PARAMETER DrsGroup
 Specify the DRS group you want to retrieve.
 
 .PARAMETER VM
 Specify the virtual machine you want to add to the DRS Group.
 
 .EXAMPLE
 Add-VMToDrsGroup -Cluster $Cluster -DrsGroup "VM DRS Group" -VM $VM
 Adds virtual machine $VM to the DRS group "VM DRS Group" of cluster $Cluster.
 
 .EXAMPLE
 Get-Cluster MyCluster | Get-VM "A*" | Add-VMToDrsGroup -Cluster MyCluster -DrsGroup $DrsGroup
 Adds all virtual machines with a name starting with "A" in cluster MyCluster to the DRS group $DrsGroup of cluster MyCluster.
 
 .INPUTS
 VirtualMachineImpl
 
 .OUTPUTS
 Task
 
 .COMPONENT
 VMware vSphere PowerCLI
 #>
 
   param([parameter(Mandatory=$true)] $Cluster,
         [parameter(Mandatory=$true)] $DrsGroup,
         [parameter(Mandatory=$true, ValueFromPipeline=$true)] $VM)
         
   begin {
     $Cluster = Get-Cluster -Name $Cluster
   }
   
   process {
     if ($Cluster) {
       if ($DrsGroup.GetType().Name -eq "string") {
         $DrsGroupName = $DrsGroup
         $DrsGroup = Get-DrsGroup -Cluster $Cluster -Name $DrsGroup
       }
       if (-not $DrsGroup) {
         Write-Error "The DrsGroup $DrsGroupName was not found on cluster $($Cluster.name)."
       }
       else { 
         if ($DrsGroup.GetType().Name -ne "ClusterVmGroup") {
           Write-Error "The DrsGroup $DrsGroupName on cluster $($Cluster.Name) doesn't have the required type ClusterVmGroup."
         }
         else {
           $VM = $Cluster | Get-VM -Name $VM
           If ($VM) {
             $spec = New-Object VMware.Vim.ClusterConfigSpecEx
             $spec.groupSpec = New-Object VMware.Vim.ClusterGroupSpec[] (1)
             $spec.groupSpec[0] = New-Object VMware.Vim.ClusterGroupSpec
             $spec.groupSpec[0].operation = "edit"
             $spec.groupSpec[0].info = $DrsGroup
             $spec.groupSpec[0].info.vm += $VM.ExtensionData.MoRef
 
             $Cluster.ExtensionData.ReconfigureComputeResource_Task($spec, $true)
           }
         }
       }
     }
   }
 }
`

