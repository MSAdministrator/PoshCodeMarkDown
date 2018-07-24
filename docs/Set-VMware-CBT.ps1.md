---
Author: justin grote
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5675
Published Date: 2015-01-09t08
Archived Date: 2015-01-15t20
---

# set vmware cbt - 

## Description

this function will enable or disable cbt on a vm, and optionally apply the change by stunning the vm with a snapshot. it is useful when cbt needs to be reset or disabled en masse.

## Comments



## Usage



## TODO



## function

`load-powerclisnapin`

## Code

`#
 #
 function Load-PowerCLISnapin{
     if (!(Get-PSSnapin VMware.vimautomation.core -erroraction SilentlyContinue)) {
         if (!(Get-pssnapin -registered vmware.vimautomation.core)) {
             write-error "VMware PowerCLI not installed. Please install VMware PowerCLI and try again"
             exit
         }
         Add-PSSnapin vmware.vimautomation.core
 
     }
 }
 
 function Set-VMChangedBlockTracking {
 <#
 .SYNOPSIS
 Enables or disables Changed Block Tracking on a Virtual Machine
 .DESCRIPTION
 This function will enable or disable CBT on a VM, and optionally apply the change by stunning the VM with a snapshot. It is useful when CBT needs to be reset or disabled en masse. 
 
 .EXAMPLE 
 Disable CBT on virtual machine and apply it, confirming the action first.
 
 set-vmchangedblocktracking TESTVM01 -enabled:$false -applynow -confirm:$true
 
 .EXAMPLE 
 Get all virtual machines with CBT disabled, enable it, and apply it.
 
 Get-VM | where {$_.ExtensionData.config.ChangeTrackingEnabled} | Set-VMChangedBlockTracking -enabled:$true -applynow
 
 .LINK
 http://www.robertwmartin.com/?p=274
 .PARAMETER VMNames
 The Virtual Machines to set CBT on. Accepts Virtual Machine Objects or one or more virtual machine names.
 .PARAMETER Enabled
 Specifies whether to enable or disable CBT. CBT does not apply until a VM stun action such as a snapshot or restart occurs unless the -applynow option is specified
 .PARAMETER ApplyNow
 Specifies whether to enable CBT immediately stun the VM by creating and subsequently deleting a VM snapshot on the VM 
 #>
 
 [CmdletBinding(SupportsShouldProcess)]
 
 Param(
     [Parameter(Mandatory=$true,ValueFromPipeline=$True,Position=0)]
     [string[]]$VMNames,
     
     [Parameter(Mandatory=$true)]
     [Boolean] $Enabled,
 
     [Parameter(Mandatory=$false)]
     [Switch] $ApplyNow
 )
 
 BEGIN {
     if (!(Get-PSSnapin VMware.vimautomation.core -erroraction SilentlyContinue)) {
         if (!(Get-pssnapin -registered vmware.vimautomation.core)) {
             write-error "VMware PowerCLI not installed. Please install VMware PowerCLI and try again"
             exit
         }
         Add-PSSnapin vmware.vimautomation.core
     }
     
     $spec = New-Object VMware.Vim.VirtualMachineConfigSpec 
     $spec.ChangeTrackingEnabled = $Enabled
 }
 
 PROCESS {
     foreach ($VMName in $VMNames) {
         $vm = get-vm $VMName
 
         if ($Enabled) {$CBTAction = "Enable"} else {$CBTAction = "Disable"}
 
         if($PSCmdlet.ShouldProcess($VMName,"$CBTAction Changed Block Tracking")) {
 
         write-verbose "Apply CBT Configuration: $($Enabled) to $VMName"
         $vm.ExtensionData.ReconfigVM($spec)
 
         
         if ($ApplyNow) {
             write-verbose "Applying CBT by stunning $($vm.ToString())"
             $snap=$vm | New-Snapshot -Name 'Disable CBT Temporary' 
             $snap | Remove-Snapshot -confirm:$false
         }
         }
     }
 
 }
 
 }
`

