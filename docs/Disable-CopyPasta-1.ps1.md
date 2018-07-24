---
Author: cody bunch
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 1475
Published Date: 2010-11-18t06
Archived Date: 2013-09-28t00
---

# disable-copypasta-1.ps1 - 

## Description

updated

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 Begin {
 
         $disableCopy = "isolation.tools.copy.enable"
         $disableCopy_value = "false"
         $disablePaste = "isolation.tools.paste.enable"
         $disablePaste_value = "false"
         $disableGUI = "isolation.tools.setGUIOptions.enable"
         $disableGUI_value = "false"
 }
 
 Process {
         if ( $_ -isnot [VMware.VimAutomation.Client20.VirtualMachineImpl] ) { co ntinue  }
 
         $vm = Get-View $_.Id
         $vmConfigSpec = New-Object VMware.Vim.VirtualMachineConfigSpec
         $vmConfigSpec.extraconfig += New-Object VMware.Vim.optionvalue
         $vmConfigSpec.extraconfig += New-Object VMware.Vim.optionvalue
         $vmConfigSpec.extraconfig += New-Object VMware.Vim.optionvalue
         $vmConfigSpec.extraconfig[0].Key=$disableCopy
         $vmConfigSpec.extraconfig[0].Value=$disableCopy_value
         $vmConfigSpec.extraconfig[1].Key=$disablePaste
         $vmConfigSpec.extraconfig[1].Value=$disablePaste_value
         $vmConfigSpec.extraconfig[2].Key=$disableGUI
         $vmConfigSpec.extraconfig[2].Value=$disableGUI_value
         $vm.ReconfigVM($vmConfigSpec)
 }
`

