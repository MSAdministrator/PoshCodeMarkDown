---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.30
Encoding: ascii
License: cc0
PoshCode ID: 1163
Published Date: 
Archived Date: 2009-06-27t19
---

#  - 

## Description

powershell script for shrinking a partition, works with vista and server 2008.

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 $DriveLetter = "D:"
 $FileLocation = $env:Temp
 $FileName4DiskPart = "Shrink.txt"
 
 $DiskDrive = GWMI -CL Win32_LogicalDisk | Where {$_.DeviceId -Eq "$DriveLetter"}
 $DriveSize = ($DiskDrive.Size /1GB)
 $DriveSize = [int]$DriveSize
 
 $DesiredShrink = [int]($DriveSize * $MaxShrink)
 $MinimumShrink = [int]($DriveSize * $MinShrink)
 
 
 Write "Select volume $DriveLetter" |Out-file $FileLocation\$FileName4DiskPart -encoding ASCII
 Write "shrink desired=$DesiredShrink minimum=$MinimumShrink" |Out-file $FileLocation\$FileName4DiskPart -append -encoding ASCII
 
 &cmd " /c diskpart" " /s $FileLocation\$FileName4DiskPart"
`

