---
Author: dollarunderscore
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6304
Published Date: 2016-04-15t07
Archived Date: 2016-04-18t21
---

# get-vmsnapshotinformatio - 

## Description

this function retrieves information about the owner/creator of a snapshot in vmware/vsphere along with some other properties.

## Comments



## Usage



## TODO



## function

`get-vmsnapshotinformation`

## Code

`#
 #
 #======================================================================================================
 #
 #======================================================================================================
 
 function Get-VMSnapshotInformation
 {
     <#
     .Synopsis
        Retrieves information about snapshots on a VM.
 
     .DESCRIPTION
        This function retrieves information about snapshots, including information on who created the snapshot and when.
        The creator information is not available in the "Get-Snapshot"-cmdlet.
 
     .EXAMPLE
        Get-VMSnapshotInformation -VM MyVirtualMachine
 
     .EXAMPLE
        Get-VM | Get-VMSnapshotInformation
 
     .PARAMETER VM
     The name of a VM to check for snapshots.
 
     #>
 
     [CmdletBinding()]
     Param
     (
         [Parameter(Mandatory=$true,
                    ValueFromPipeline=$true,
                    ValueFromPipelineByPropertyName=$true,
                    HelpMessage="Enter the name of a VM")]
         [Alias('Name')]
         $VM)
 
     Begin { }
 
     Process
     {
         foreach ($VirtualMachine in $VM) {
             $SnapShots = $null
             $SnapShots = Get-Snapshot $VirtualMachine
 
             if ($SnapShots -eq $null) {
                 Continue
             }
 
             foreach ($SnapShot in $SnapShots) {
                 $TaskMgr = Get-View TaskManager
 
                 $Filter = New-Object VMware.Vim.TaskFilterSpec
 
                 $Filter.Time = New-Object VMware.Vim.TaskFilterSpecByTime
                 $Filter.Time.beginTime = ((($Snapshot.Created).AddSeconds(-5)).ToUniversalTime())
                 $Filter.Time.timeType = "startedTime"
                 $Filter.Time.EndTime = ((($Snapshot.Created).AddSeconds(5)).ToUniversalTime())
 
                 $Filter.State = "success"
 
                 $Filter.Entity = New-Object VMware.Vim.TaskFilterSpecByEntity
                 $Filter.Entity.recursion = "self"
                 $Filter.Entity.entity = (Get-Vm -Name $Snapshot.VM.Name).Extensiondata.MoRef
 
                 $TaskCollector = Get-View ($TaskMgr.CreateCollectorForTasks($Filter))
 
                 $TaskCollector.RewindCollector | Out-Null
 
                 $Tasks = $TaskCollector.ReadNextTasks(100)
         
                 $SnapUser = $null
 
                 foreach ($Task in $Tasks) {
                     $GuestName = $Snapshot.VM
                     $Task = $Task | where {$_.DescriptionId -eq "VirtualMachine.createSnapshot" -and $_.State -eq "success" -and $_.EntityName -eq $GuestName}
 
                     If ($Task -ne $null) {
                         $SnapUser = $Task.Reason.UserName
                     }
                 }
 
                 $TaskCollector.DestroyCollector()
 
                 $returnObject = New-Object System.Object
                 $returnObject | Add-Member -Type NoteProperty -Name VMName -Value $SnapShot.VM.Name
                 $returnObject | Add-Member -Type NoteProperty -Name Size -Value $SnapShot.SizeMB
                 $returnObject | Add-Member -Type NoteProperty -Name Name -Value $SnapShot.Name
                 $returnObject | Add-Member -Type NoteProperty -Name Id -Value $SnapShot.Id
                 $returnObject | Add-Member -Type NoteProperty -Name Creator -Value $SnapUser
                 $returnObject | Add-Member -Type NoteProperty -Name CreatedTime -Value $SnapShot.Created
 
                 Write-Output $returnObject
             }
         }
     }
 
     End { }
 }
`

