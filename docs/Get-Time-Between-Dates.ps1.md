---
Author: dan in philly
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: utf-8
License: cc0
PoshCode ID: 6889
Published Date: 2017-05-08t04
Archived Date: 2017-05-13t18
---

# get time between dates - 

## Description

provide a begin and end time frame (mm dd yyyy) and this will calculate the years, months and days between the two dates.  this is not 100% accurate but it was close enough for what i was trying to do.  someone with better math skills should be able to improve the accuracy.

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 Get-AppxProvisionedPackage -Online | Where {$_.PackageName -notlike �*store*� -and $_.PackageName -notlike �*calc*�} | Remove-AppxProvisionedPackage -Online
 Get-AppxPackage | Where {$_.PackageFullName -notlike �*store*� -and $_.PackageFullName -notlike �*calc*�} | Remove-AppxPackage
`

