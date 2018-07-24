---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 4574
Published Date: 
Archived Date: 2016-12-25t13
---

#  - 

## Description

app-v 5.0, create friendly folder names for packages. requires powershell community extensions.

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 $FriendlyFolderName = "MyFriendlyAppV"
 $appvroot = $(Get-Itemproperty HKLM:\SOFTWARE\Microsoft\AppV\Client\Streaming).PackageInstallationRoot
 $appvPSroot = $appvroot.Replace('%programdata%',$env:ProgramData)
 
 Get-AppvClientPackage | ForEach-Object {
     $targetpath = $appvPSroot + '\' + $_.PackageID.ToString() + '\' + $_.VersionID.ToString()
 	$Path = "C:\ProgramData\" + $FriendlyFolderName + "\" + $_.Name
 	New-Junction -LiteralPath $Path -TargetPath $targetpath
 }
`

