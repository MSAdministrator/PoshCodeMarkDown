---
Author: ed goad
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 3392
Published Date: 2012-05-02t13
Archived Date: 2012-05-10t14
---

# set resource limits - 

## Description

set the resource limits on vms automatically based on the number of vcpus and vram assigned. this is useful as a preliminary pass at resource tiering.

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 Param([Parameter(Mandatory=$true)] [string]$VMGuest)
 
 $vm = get-vm $VMGuest
 $cpuCap = $vm.NumCPU*1000
 $cpuRes = $cpuCap/2
 
 $memCap = $vm.MemoryMB
 $memRes = $memCap/4
 
 $spec = new-object VMware.Vim.VirtualMachineConfigSpec;
 $spec.memoryAllocation = New-Object VMware.Vim.ResourceAllocationInfo;
 $spec.memoryAllocation.Shares = New-Object VMware.Vim.SharesInfo;
 $spec.memoryAllocation.Limit = -1;
 $spec.memoryAllocation.Reservation = $memRes;
 
 $spec.cpuAllocation = New-Object VMware.Vim.ResourceAllocationInfo;
 $spec.cpuAllocation.Limit = $cpuCap;
 $spec.cpuAllocation.Reservation = $cpuRes;
 
 ($vm | get-view).ReconfigVM_Task($spec)
`

