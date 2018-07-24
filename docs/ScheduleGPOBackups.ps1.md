---
Author: clint
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 3243
Published Date: 2012-02-18t23
Archived Date: 2012-02-23t05
---

# schedulegpobackups.ps1 - 

## Description

requires powershell 2.0 & the grouppolicy module.

## Comments



## Usage



## TODO



## module

``

## Code

`#
 #
 Import-Module grouppolicy
 $domain = "mydomain.com"
 $gpoBackupRootDir = "c:\gpoBackups"
 $backupDir = "$gpoBackupRootDir\{0:yyyy-MM}" -f (Get-Date)
 
 #$fullBackupFrequency = "Day"
 #$fullBackupFrequency = "Week"
 $fullBackupFrequency = "Month"
 #$fullBackupFrequency = "Year"
 
 $IncBackupFreqency = "Hour"
 
 $numKeepBackupSets = 12
 
 #$startOfWeek = "Sunday"
 $startOfWeek = "Monday"
 #$startOfWeek = "Tuesday"
 #$startOfWeek = "Wednesday"
 #$startOfWeek = "Thursday"
 #$startOfWeek = "Friday"
 #$startOfWeek = "Saturday"
 
 $startOfMonth = 1
 
 $startOfYear = 1
 
 
 $currentDateTime = Get-Date
 $doFull = $false
 $doInc  = $false
 
 if (-not (Test-Path $backupDir))
 {
 	try 
 	{
 		New-Item -ItemType Directory -Path $backupDir
 	}
 	catch
 	{
 		Throw $("Could not create directory $backupDir")
 	}
 }
 
 if ( Test-Path $backupDir\LastFullTimestamp.xml )
 {
 	$lastFullTimestamp = Import-Clixml $backupDir\LastFullTimestamp.xml
 	if ( $lastFullTimestamp -isnot [datetime] )
 	{
 		$doFull = $true
 		Remove-Item $backupDir\LastFullTimestamp.xml
 	}
 	{
 		$fullDelta = $currentDateTime - $lastFullTimestamp
 		switch ($fullBackupFrequency)
 		{
 			Day
 			{
 				if ( $fullDelta.days -gt 0 )
 				{
 					$doFull = $true
 				}
 			}
 			Week
 			{
 				if ( ($currentDateTime.dayOfWeek -eq [DayOfWeek]$startOfWeek) `
 					 -or ($fullDelta.days -gt 7) )
 				{
 					$doFull = $true
 				}
 			}
 			Month
 			{
 				if ( ($currentDateTime.day -eq $startOfMonth) `
 					 -or ($fullDelta.days -gt 30) )
 				{
 					$doFull = $true
 				}
 			}
 			Year
 			{
 				if ( ($currentDateTime.dayofyear -eq $startOfYear) `
 					 -or ($fullDelta.days -gt 365) )
 				{
 					$doFull = $true
 				}
 			}
 		}
 	}
 }
 {
 	$doFull = $true
 }
 
 if ($doFull)
 {
 	$GPOs = Get-GPO -domain $domain -All
 	foreach ($GPO in $GPOs)
 	{
 		$GPOBackup = Backup-GPO $GPO.DisplayName -Path $backupDir
 	    $ReportPath = $backupDir + "\" + $GPO.ModificationTime.Year + "-" + $GPO.ModificationTime.Month + "-" + $GPO.ModificationTime.Day + "_" +  $GPO.Displayname + "_" + $GPOBackup.Id + ".html"
 	    Get-GPOReport -Name $GPO.DisplayName -path $ReportPath -ReportType HTML 		
 	}
 	Export-Clixml -Path $backupDir\LastFullTimestamp.xml -InputObject ($currentDateTime)
 }
 {
 	if ( Test-Path $backupDir\LastIncTimestamp.xml )
 	{
 		$lastIncTimestamp = Import-Clixml $backupDir\LastIncTimestamp.xml
 		if ( $lastIncTimestamp -isnot [datetime] )
 		{
 			$lastFullTimestamp = Import-Clixml $backupDir\LastFullTimestamp.xml
 			$IncDelta = $currentDateTime - $lastFullTimestamp
 			$doInc = $true
 			Remove-Item $backupDir\LastIncTimestamp.xml
 		}
 		{
 			$IncDelta = $currentDateTime - $lastIncTimestamp
 		}
 	}
 	{
 		$lastFullTimestamp = Import-Clixml $backupDir\LastFullTimestamp.xml
 		$IncDelta = $currentDateTime - $lastFullTimestamp
 	}
 	if ($doInc -eq $false)
 	{
 		switch ($IncBackupFreqency)
 		{
 			Hour
 			{
 				if ($IncDelta.hours -gt 0)
 				{
 					$doInc = $true
 				}
 			}
 			Day
 			{
 				if ($IncDelta.days -gt 0)
 				{
 					$doInc = $true
 				}
 			}
 			Week
 			{
 				if ( ($currentDateTime.dayOfWeek -eq [DayOfWeek]$startOfWeek) `
 					 -or ($IncDelta.days -gt 7) )
 				{
 					$doInc = $true
 				}
 			}
 			Month
 			{
 				if ( ($currentDateTime.day -eq $startOfMonth) `
 					 -or ($IncDelta.days -gt 30) )
 				{
 					$doInc = $true
 				}
 			}
 		}
 	}
 	if ($doInc)
 	{
 		$GPOs = Get-GPO -domain $domain -All | Where-Object { $_.modificationTime -gt ($currentDateTime - $incDelta) }
 		foreach ($GPO in $GPOs)
 		{
 			$GPOBackup = Backup-GPO $GPO.DisplayName -Path $backupDir
 		    $ReportPath = $backupDir + "\" + $GPO.ModificationTime.Year + "-" + $GPO.ModificationTime.Month + "-" + $GPO.ModificationTime.Day + "_" +  $GPO.Displayname + ".html"
 		    Get-GPOReport -Name $GPO.DisplayName -path $ReportPath -ReportType HTML 		
 		}
 		Export-Clixml -Path $backupDir\LastIncTimestamp.xml -InputObject ($currentDateTime)
 	}
 }
`

