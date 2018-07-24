---
Author: trevor wilson
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5467
Published Date: 2016-09-25t18
Archived Date: 2016-06-06t19
---

# delete empty folders - 

## Description

this is a script to remove empty folders from a drive. i used it when i had to clear up a shared drive from a former company.

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 $Drive = Read-Host "Path to Folders"
 Write-Host "This will delete all empty folders in this directory!"
 $a = Get-ChildItem $drive -recurse | Where-Object {$_.PSIsContainer -eq $True}
 $a | Where-Object {($_.GetFiles().Count -lt 1 -and $_.GetDirectories().Count -lt 1)} | Select-Object FullName | ForEach-Object {remove-item $_.fullname -recurse} 
 Write-Host "All Done!"
`

