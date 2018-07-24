---
Author: yasen kalchev
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: utf-8
License: cc0
PoshCode ID: 2017
Published Date: 
Archived Date: 2010-07-26t15
---

# error handling powercli - 

## Description

error handling with powercli 4.1

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 $vmPath = "[Storage1] MyVM/MyVM.vmx"
 $vm = New-VM �VMHost "192.168.1.10" �VMFilePath $vmPath -Name MyVM
 
 if (!$?) {
     if ($error[0].Exception �is [VMware.VimAutomation.ViCore.Types.V1.ErrorHandling.DuplicateName]) {
 	$vm = New-VM �VMHost "192.168.1.10" �VMFilePath $vmPath �Name "myVM (1)"
     } elseif ($error[0].Exception �is [VMware.VimAutomation.ViCore.Types.V1.ErrorHandling.InsufficientResourcesFault]) {
 	$vm = New-VM �VMHost "192.168.1.12" �VMFilePath $vmPath
     } else {
 
     }
 }
`

