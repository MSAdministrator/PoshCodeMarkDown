---
Author: username
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6135
Published Date: 2016-12-07t23
Archived Date: 2016-03-19t00
---

# pstmapper - 

## Description

this script will help a user to batch connect multiple pst files in microsoft outlook (personal email archives).

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 
 $host.ui.RawUI.WindowTitle = "PST Mapper"
 $host.UI.RawUI.BackgroundColor = 'DarkBlue'
 $host.UI.RawUI.ForegroundColor = 'White'
 Clear-Host
 
 [Reflection.Assembly]::LoadWithPartialname("Microsoft.Office.Interop.Outlook") | Out-Null
 $outlook = New-Object -ComObject Outlook.Application
 
 [string]$rootFolder = Read-Host "`nPath where your PST files are located"
 
 $pstFiles = Get-ChildItem -Path $rootFolder -Filter *.pst -Recurse
 
 if ($pstFiles) {
     $pstFiles | sort LastWriteTime -Descending | select LastWriteTime, Fullname | ft -auto 
 } else {
     Write-Host "No pst files found" -ForegroundColor Red
     Start-Sleep -Seconds 5
     Break
 }
 
 [datetime]$cutoff = Read-Host "Only map files newer than (date)"
 
 Write-Host "`nMapping pst files now.  This window will automatically close when the process is complete"
 
 $pstFiles | where { $_.LastWriteTime -ge $cutoff } | ForEach-Object { $outlook.Session.AddStore($_.FullName) }
`

