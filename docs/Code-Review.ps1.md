---
Author: albvar
Publisher: 
Copyright: 
Email: 
Version: 1.0
Encoding: ascii
License: cc0
PoshCode ID: 5885
Published Date: 2015-06-06t19
Archived Date: 2015-06-11t14
---

# code review - 

## Description

deletes old shadow copies – runs as a scheduled task. looking for a code review to improve upon what i have learned from the community.

## Comments



## Usage



## TODO



## script

`get-shadowcopy`

## Code

`#
 #
 <#
         .SYNOPSIS
         Scan the entire environment for stale shadow copies and delete
 
         .DESCRIPTION
         Will run on [TODO] as a scheduled task on a [TODO] minute interval.
 
         .NOTES  
 
         HISTORY
         -------
         6/01/2015 V1.0 - Added Get-ShadowCopy
         6/02/2015 V1.1 - Added Get-ShadowCopyDiskUsage
         6/05/2015 V1.2 - Added Remove-ShadowCopy
         2. Added progress bar to track deletion
         3. Added logic to calculate free space after shadow copies have been removed
     
 
     
         REQUIREMENTS
         ------------
         1. Requires that the account running the script has admin rights to invoke the .Delete method from Win32_ShadowCopy
         2. Requires ActiveDirectory module to build dynamic list of servers to search against
         3. Requires Powershell v3 for on demand module loading
 
         .EXAMPLE
         powershell.exe -NoProfile -ExecutionPolicy ByPass -File Remove-StaleVSSCopies.ps1
 
         .OUTPUT
         ComputerName        : COMPUTER1234
         OriginatingVolume   : R:\
         DiffVolume          : D:\
         DiffVolumeFreeGB    : 33.52
         VSSDiskUsedGB       : 1.74
         VSSCount            : 3
         SpaceRecoveredGB    : 1.74
         NewDiffVolumeFreeGB : 35.52
 
         .LINKS
         Win32_ShadowCopy class https://msdn.microsoft.com/en-us/library/aa394428%28v=vs.85%29.aspx
         To manually create a shadow copy: vssadmin create shadow /for=c:
 #>
 
 
 function Get-ShadowCopy 
 {
     <#
             .Synopsis
             Identify old shadow copies on a remote system
 
             .Output 
             System.Management.ManagementBaseObject
             .Example
             Get-OldShadowCopies -ComputerName Computer1 -MinimumElapsedDays 60
 
             Returns shadow copies that are at least 60 days old from todays date
             .Example
             'Computer1', 'Computer2' | Get-OldShadowCopies -MinimumElapsedDays 20
 
             Returns shadow copies that are at least 20 days old from todays date
     #>
     [CmdletBinding()]
     [OutputType('ManagementBaseObject')]
     param(
         [Parameter(Mandatory = $true,
                 ValueFromPipeLine = $true,
         Position = 0)]
         [string]
         $ComputerName,
         [Parameter(Mandatory = $false,
                 ValueFromPipeLine = $true,
         Position = 1)]
         [ValidateScript({
                     $_ -gt 3 
         })]
         [int]$MinimumElapsedDays = 30
     )
     begin {
         if ($ComputerName.Count -gt 0) 
         {
             $stopWatch = [System.Diagnostics.Stopwatch]::StartNew() 
         }
     }
     process {   
         Write-Verbose -Message ([String]::Format('Starting Get-ShadowCopy on {0} - only shadow copies that are at least {1} days old are actionable', $ComputerName, $MinimumElapsedDays))
         foreach ($computer in $ComputerName) 
         {
             try 
             {
                 $shadowCopy = Get-WmiObject -Class Win32_ShadowCopy -ComputerName $computer -ErrorAction Stop
                 if ($shadowCopy.Count -gt 0) 
                 {            
                     $countActionableShadowCopies = 0
                     $shadowCopy | ForEach-Object -Process {
                         $snapshotDate = [Management.ManagementDateTimeConverter]::ToDateTime($_.InstallDate)
                         $elapsedDays = (New-TimeSpan $snapshotDate (Get-Date)).Days
                         Write-Verbose -Message "$($_.ID) : $elapsedDays days old"
                         if ($elapsedDays -gt $MinimumElapsedDays) 
                         {
                             $countActionableShadowCopies++
                             $_ | Add-Member -NotePropertyName VSSCounts -NotePropertyValue $shadowCopy.Count -PassThru
                         }
                     }
                     Write-Verbose -Message ([String]::Format('Total shadow copies identified on {0} {1} only {2} are at least {3} days old', 
                     $ComputerName, $($shadowCopy.Count), $countActionableShadowCopies, $MinimumElapsedDays))
                 }
             }
             catch 
             {
                 'Could not retrieve shadowcopy on {0} Error{1}' -f $ComputerName, $_.Exception
             }
         }
     }   
     end {
         if ($ComputerName.Count -gt 0) 
         {
             Write-Verbose -Message "`tCompleted Get-ShadowCopy in $($stopWatch.Elapsed)" 
         }
     }
 }
 
 function Get-ShadowCopyDiskUsage 
 {
     <#
             .Synopsis
             Determine how much space is being taken up by a shadow copy
     #>
     [CmdletBinding()]
     [OutputType([String])]
     param(
         [Parameter(Mandatory = $true,
                 ValueFromPipeLine = $true,
         Position = 0)]
         [System.Management.ManagementBaseObject]
         $shadowCopy
     )     
     begin {
         $stopWatch = [System.Diagnostics.Stopwatch]::StartNew()
     }
     process {
         foreach ($snapshot in $shadowCopy) 
         {
             try 
             {  
                 $snapshotVolumeGUID = $snapshot.VolumeName.Replace('\\?\Volume','').Replace('{','').Replace('}\','')
                 $wmiCommonParams = @{
                     ComputerName = $snapshot.PSComputerName
                     Class        = 'Win32_ShadowStorage'
                     Property     = 'AllocatedSpace', 'DiffVolume', 'MaxSpace', 'UsedSpace', 'Volume'
                     ErrorAction  = 'Stop'
                 }
                 $shadowStorage = Get-WmiObject @wmiCommonParams | Where-Object {
                     $_.Volume -like "*$snapshotVolumeGUID*"
                 }
                 $shadowStorageProp = '' | Select-Object  ComputerName, OriginatingVolume, DiffVolume, DiffVolumeFreeGB, VSSDiskUsedGB, VSSCount
                 $shadowStorageProp.ComputerName = $snapshot.PSComputerName
                 $queryOriginatingVolume = ([String]::Format('\\{0}\root\cimv2:{1}', $snapshot.PSComputerName, $shadowStorage.Volume))
                 $queryDiffVolume = ([String]::Format('\\{0}\root\cimv2:{1}', $snapshot.PSComputerName, $shadowStorage.DiffVolume))
                 $shadowStorageProp.DiffVolume = $diffVolume.Name
                 $shadowStorageProp.DiffVolumeFreeGB = [math]::Round(($diffVolume.FreeSpace/1GB),2)
                 $shadowStorageProp.OriginatingVolume = $originatingVolume.Name   
                 $shadowStorageProp.VSSDiskUsedGB = [math]::Round(($shadowStorage.AllocatedSpace/1GB),2)
                 $shadowStorageProp.VSSCount = $snapshot.VSSCounts
                 $shadowStorageProp            
             }
             catch 
             {
                 Write-Verbose "Unable to query $($snapshot.PSComputerName) Error:"
                 Write-Verbose "Error Command = $queryDiffVolume"            
             }
         }
     }
     end {
         if ($shadowCopy.Count -gt 0) 
         {
             Write-Verbose "`tCompleted Get-ShadowCopyDiskUsage in $($stopWatch.Elapsed)" 
         }
     }
 }                                       
 function Remove-ShadowCopy 
 {
     <#
             .Synopsis
             Delete shadow copy
     #>
     [CmdletBinding()]
     param(
         [Parameter(Mandatory = $true,
                 ValueFromPipeLine = $true,
         Position = 0)]
         [System.Management.ManagementBaseObject]
         $snapshot,
         [switch]$Remove
     )
     begin {
         $stopWatch = [System.Diagnostics.Stopwatch]::StartNew()
         [int]$count = 0
     }
     process {
         foreach ($object in $snapshot) 
         {
             $count++
             Write-Warning "Deleting $($object.ID) from $($object.PSComputerName)"
             Write-Progress -Activity "Deleting snapshots from $($object.PSComputerName)" `
              -Status "Processing snapshot: $count of $($allVSSItems.Count)" -Id 1 -PercentComplete ($count/$allVSSItems.Count*100)
             if ($Remove) 
             {
                 $object.Delete() 
             }
         }
     }
     end {
         if ($snapshot.Count -gt 0) 
         {
             Write-Verbose "`tCompleted Remove-ShadowCopy in $($stopWatch.Elapsed)" 
         }
     }    
 }
 
 $allVSSItems = 'COMPUTER1234' | Get-ShadowCopy -MinimumElapsedDays 30 -Verbose
 $uniqueVSSVolumes = $allVSSItems | Sort-Object -Property VolumeName -Unique
 $diskUsage = $uniqueVSSVolumes | Get-ShadowCopyDiskUsage -Verbose 
 $diskUsage | Sort-Object -Property ComputerName
 $allVSSItems | Remove-ShadowCopy -Verbose
 
 $results = @()
 foreach ($obj in $diskUsage) 
 {
     $diffVolume = $obj.DiffVolume
     if ($diffVolume[-1] -ne '\') 
     {
         $diffVolume += '\' 
     }
     
     $wmiParams = @{
         ComputerName = $obj.ComputerName
         Query        = "Select FreeSpace From Win32_Volume Where Name='$diffVolume'"
     }
     $freeSpace = [math]::Round(((Get-WmiObject @wmiParams).FreeSpace / 1GB),2)
     $spaceRecovered = $freeSpace - $obj.DiffVolumeFreeGB
     
     $newList = $obj | Select-Object *, SpaceRecoveredGB, NewDiffVolumeFreeGB
     $newList.SpaceRecoveredGB = $spaceRecovered
     $newList.NewDiffVolumeFreeGB = $freeSpace
     $results += $newList
 }
 
 $results | Out-GridView
 
 
 
 #>
`

