---
Author: dragonmc77
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5240
Published Date: 2015-06-14t05
Archived Date: 2015-01-31t20
---

# get-lastmoddate - 

## Description

reports the latest lastmodified date of all the files within a folder.

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 param(  [string]$Path,
         [string]$LogFile = "")
 
 $FolderList = Get-ChildItem $Path | Where-Object {$_.PSIsContainer}
 
 
 foreach ($Folder in $FolderList) {
     if ($Folder.Name.ToLower() -eq "dfsrprivate") {continue}
     $LatestDate = Get-Date -Date 1/1/1940
     $Output = New-Object Object | Select LastModDate,Path
     $Output.Path = $Folder.FullName
     Get-ChildItem $Folder.Fullname -Recurse | 
         Where-Object {$_.GetType().ToString() -eq "System.IO.FileInfo"} |
         ForEach-Object {
             if ($_.LastWriteTime -gt $LatestDate) {$LatestDate = $_.LastWriteTime}
     }
 	if ($LatestDate -eq (Get-Date -Date 1/1/1940)) {$LatestDate = $Folder.LastWriteTime}
     $Output.LastModDate = $LatestDate
     Write-Output $Output
     if ($LogFile -ne "") {
         Add-Content -Path $LogFile -Value "$($Output.LastModDate)   $($Output.Path)" -Force
     }
 }
`

