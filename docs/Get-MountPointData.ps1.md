---
Author: joakim svendsen
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 3717
Published Date: 2013-10-27t11
Archived Date: 2013-07-21t00
---

# get-mountpointdata - 

## Description

get information about free/used space on windows server mount points (regular drives are also supported) using get-mountpointdata — click that link for more information.

## Comments



## Usage



## TODO



## function

`get-deviceidfrommp`

## Code

`#
 #
 function Get-DeviceIDFromMP {
     
     param([Parameter(Mandatory=$true)][string] $VolumeString,
           [Parameter(Mandatory=$true)][string] $Directory)
     
     if ($VolumeString -imatch '^\s*Win32_Volume\.DeviceID="([^"]+)"\s*$') {
         $Matches[1] -replace '\\{2}', '\'
     }
     else {
         "Unknown device ID for " + $Directory
     }
     
 }
 
 function Get-MountPointData {
     
     param([Parameter(Mandatory=$true)][string[]] $ComputerName,
           #[switch] $DoNotExcludeDefaults,
           [switch] $IncludeRootDrives
           )
     
     foreach ($Computer in $ComputerName) {
         
         try {
             
             $MountPointData = @{}
             Get-WmiObject Win32_MountPoint -ComputerName $Computer | 
                 Where { if ($IncludeRootDrives) { $true } else { $_.Directory -NotMatch '^\s*Win32_Directory\.Name="[a-z]:\\{2}"\s*$' } } | ForEach-Object {
                     $MountPointData.(Get-DeviceIDFromMP $_.Volume $_.Directory) = $_.Directory
             }
             
             $Volumes = Get-WmiObject Win32_Volume -ComputerName $Computer | Where {
                     if ($IncludeRootDrives) { $true } else { -not $_.DriveLetter }
                 } | 
                 Select-Object Label, Caption, Capacity, FreeSpace, FileSystem, DeviceID, @{n='Computer';e={$Computer}}
             
         }
         
         catch {
             
             Write-Host -Fore Red "Terminating WMI error for ${Computer} (skipping): $($Error[0])"
             continue
             
         }
         
         if (-not $Volumes) {
             Write-Host -Fore Red "No mount points found on $Computer. Skipping..."
             continue
         }
         
         $Volumes | ForEach-Object {
             
             if ($MountPointData.ContainsKey($_.DeviceID)) {
                 
                 if ($_.Capacity) { $PercentFree = $_.FreeSpace*100/$_.Capacity }
                 else { $PercentFree = 0 }
                 
                 $_ | Select-Object Computer, Label, Caption, FileSystem, @{n='Size (GB)';e={$_.Capacity/1GB}},
                     @{n='Free space';e={($_.FreeSpace/1GB).ToString('N')}}, @{n='Percent free';e={$PercentFree}}
                 
             }
             
         } | Sort-Object -Property 'Percent free', @{Descending=$true;e={$_.'Size (GB)'}}, Label, Caption |
             Select-Object Computer, Label, Caption, FileSystem, @{n='Size (GB)';e={$_.'Size (GB)'.ToString('N')}},
                           'Free space', @{n='Percent free';e={$_.'Percent free'.ToString('N')}}
         
     }
     
 }
`

