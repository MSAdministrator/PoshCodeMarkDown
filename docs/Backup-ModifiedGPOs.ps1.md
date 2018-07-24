---
Author: jan egil ring
Publisher: 
Copyright: 
Email: jan.egil.ring@powershell.no
Version: 1.2
Encoding: ascii
License: cc0
PoshCode ID: 2400
Published Date: 
Archived Date: 2010-12-10t22
---

# backup-modifiedgpos.ps1 - backup-modifiedgpos.ps1

## Description

all group policy objects modified in the specified timespan are backup up to the specified backup path.

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 ###########################################################################"
 #
 #
 #
 #
 #
 #
 #
 ###########################################################################"
 
 
 Param([switch]$FirstRun)
 
 Import-Module grouppolicy
 
 $BackupPath = "C:\GPO Backup"
 $ReportPath = "C:\GPO Backup\Reports\"
 if (!(Test-Path -path $BackupPath)) {New-Item $BackupPath -type directory}
 if (!(Test-Path -path $ReportPath)) {New-Item $ReportPath -type directory}
 
 if ($FirstRun) {
 $Timespan = (Get-Date).AddDays(-5000)
 }
 else {
 $Timespan = (Get-Date).AddDays(-1)
 }
 
 $ModifiedGPOs = Get-GPO -all | Where-Object {$_.ModificationTime -gt $Timespan}
 
 if ($ModifiedGPOs) {
 
 Write-Host "The following "$ModifiedGPOs.count" GPOs were successfully backed up:" -ForegroundColor yellow
 
 Foreach ($GPO in $ModifiedGPOs) { 
 
     $GPOBackup = Backup-GPO $GPO.DisplayName -Path $BackupPath
     $Path = $ReportPath + $GPO.ModificationTime.Month + "-"+ $GPO.ModificationTime.Day + "-" + $GPO.ModificationTime.Year + "_" +  
 
 $GPO.Displayname + "_" + $GPOBackup.Id + ".html" 
     Get-GPOReport -Name $GPO.DisplayName -path $Path -ReportType HTML
 
     Write-Host $GPO.DisplayName
 }
 }
 else {
 Write-Host "No GPO`s modified since $Timespan. If this is your initial backup, run the script with the -FirstRun switch parameter to backup all GPO`s" -ForegroundColor yellow
 }
`

