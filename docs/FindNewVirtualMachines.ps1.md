---
Author: glnsize
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 1084
Published Date: 2010-05-08t11
Archived Date: 2017-03-20t23
---

# findnewvirtualmachines - 

## Description

script to find any vm’s that we’re added to vcenter in the last x days.

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 Param(
     [int]
     $LastDays
 )
 Process
 {
     $EventFilterSpecByTime = New-Object VMware.Vim.EventFilterSpecByTime
     If ($LastDays)
     {
         $EventFilterSpecByTime.BeginTime = (get-date).AddDays(-$($LastDays))
     }
     $EventFilterSpec = New-Object VMware.Vim.EventFilterSpec
     $EventFilterSpec.Time = $EventFilterSpecByTime
     $EventFilterSpec.DisableFullMessage = $False
     $EventFilterSpec.Type = "VmCreatedEvent","VmDeployedEvent","VmClonedEvent","VmDiscoveredEvent","VmRegisteredEvent"
     $EventManager = Get-View EventManager
     $NewVmTasks = $EventManager.QueryEvents($EventFilterSpec)
 
     Foreach ($Task in $NewVmTasks)
     {
         If ($Task.Template -and $Task.SrcTemplate.Vm)
         {
             
             $srcTemplate = Get-View $Task.SrcTemplate.Vm -Property name | 
                 Select -ExpandProperty Name
         }
         Else
         {
             $srcTemplate = $null
         }
         write-output ""|Select-Object @{
                 Name="Name"
                 Expression={$Task.Vm.name}
             }, @{
                 Name="Created"
                 Expression={$Task.CreatedTime}
             }, @{
                 Name="UserName"
                 Expression={$Task.UserName}
             }, @{        
                 Name="Type"
                 Expression={$Task.gettype().name}
             }, @{
                 Name="Template"
                 Expression={$srcTemplate}
             }
     } 
 }
`

