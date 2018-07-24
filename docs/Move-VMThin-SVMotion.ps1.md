---
Author: jgrote
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 1579
Published Date: 2010-01-15t10
Archived Date: 2016-09-09t23
---

# move-vmthin svmotion - 

## Description

a powershell module to perform storage vmotions from thick-to-thin. meant to be used in place of move-vm. currently only accepts one vm and strings for performance reasons, will accept objects in next revision as well as more documentation.

## Comments



## Usage



## TODO



## function

`move-vmthin`

## Code

`#
 #
 function Move-VMThin {
     PARAM(
          [Parameter(Mandatory=$true,ValueFromPipeline=$true,HelpMessage="Virtual Machine Objects to Migrate")]
          [ValidateNotNullOrEmpty()]
             [System.String]$VM
         ,[Parameter(Mandatory=$true,HelpMessage="Destination Datastore")]
          [ValidateNotNullOrEmpty()]
             [System.String]$Datastore
     )
     
 	Begin {
     
     Process {        
         $vmView = Get-View -ViewType VirtualMachine -Filter @{"Name" = "$VM"}
         $dsView = Get-View -ViewType Datastore -Filter @{"Name" = "$Datastore"}
         
         if (($dsView.info.freespace / 1GB) -lt 50) {throw "Move-ThinVM ERROR: Destination Datastore $Datastore has less than 50GB of free space. This script requires at least 50GB of free space for safety. Please free up space or use the VMWare Client to perform this Migration"}
 
         $spec = New-Object VMware.Vim.VirtualMachineRelocateSpec
         $spec.datastore =  $dsView.MoRef
         $spec.transform = "sparse"
         
         $vmView.RelocateVM($spec, $null)
 }
`

