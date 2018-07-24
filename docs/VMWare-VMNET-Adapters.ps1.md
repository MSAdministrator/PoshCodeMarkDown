---
Author: oisin grehan
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 1505
Published Date: 2010-12-02t20
Archived Date: 2015-07-30t00
---

# vmware vmnet adapters - 

## Description

are you sick and tired of switching the vmware virtual network adapters from the �public� network profile to a �private� network profile each time you reboot or do something else that causes vista/win2008 to forget what you told it to do ten minutes ago?

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 #
 #
 #
 #
 #
 
 
 $identity = [Security.Principal.WindowsIdentity]::GetCurrent()
 $principal = new-object Security.Principal.WindowsPrincipal $identity
 $elevated = $principal.IsInRole([Security.Principal.WindowsBuiltInRole]::Administrator)
 
 if (-not $elevated) {
     $error = "Sorry, you need to run this script"
     if ([System.Environment]::OSVersion.Version.Major -gt 5) {
         $error += " in an elevated shell."
     } else {
         $error += " as Administrator."
     }
     throw $error
 }
 
 function confirm {
     $host.ui.PromptForChoice("Continue", "Process adapter?",
         [Management.Automation.Host.ChoiceDescription[]]@("&No", "&Yes"), 0) -eq $true
 }
 
 pushd 'hklm:\SYSTEM\CurrentControlSet\Control\Class\{4D36E972-E325-11CE-BFC1-08002BE10318}'
 
 dir -ea 0  | % {
     $node = $_.pspath
     $desc = gp $node -name driverdesc
     if ($desc -like "*vmware*") {
         write-host ("Found adapter: {0} " -f $desc.driverdesc)
         if (confirm) {
             new-itemproperty $node -name '*NdisDeviceType' -propertytype dword -value 1
         }
     }
 }
 popd
 
 gwmi win32_networkadapter | ? {$_.name -like "*vmware*" } | % {
     
     write-host -nonew "Disabling $($_.name) ... "
     $result = $_.Disable()
     if ($result.ReturnValue -eq -0) { write-host " success." } else { write-host " failed." }
     
     write-host -nonew "Enabling $($_.name) ... "
     $result = $_.Enable()
     if ($result.ReturnValue -eq -0) { write-host " success." } else { write-host " failed." }
 }
`

