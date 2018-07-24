---
Author: rob sewell http
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: utf-8
License: cc0
PoshCode ID: 4434
Published Date: 2015-09-01t11
Archived Date: 2015-07-09t03
---

# show drive sizes - show-drivesizes.ps1

## Description

#############################################################################################

## Comments



## Usage

show-drivesizes server1

## TODO



## function

`show-drivesizes`

## Code

`#
 #
 
  #############################################################################################
 #
 #
 
 
 Function Show-DriveSizes ([string]$Server)
 {
                             $Date = Get-Date
                             Write-Host -foregroundcolor DarkBlue -backgroundcolor yellow "$Server - - $Date"
                             $disks=Get-WmiObject -Class Win32_logicaldisk -Filter "Drivetype=3" -ComputerName $Server
                             $diskData=$disks | Select DeviceID, VolumeName , 
                             @{Name="SizeGB";Expression={$_.size/1GB -as [int]}},
                             @{Name="FreeGB";Expression={"{0:N2}" -f ($_.Freespace/1GB)}},
                             @{Name="PercentFree";Expression={"{0:P2}"  -f ($_.Freespace/$_.Size)}}
                             $diskdata | Format-table -AutoSize | Out-Host
                                                   
 }
`

