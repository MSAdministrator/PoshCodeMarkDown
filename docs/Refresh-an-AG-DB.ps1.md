---
Author: rob sewell http
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: utf-8
License: cc0
PoshCode ID: 5412
Published Date: 2016-09-08t19
Archived Date: 2016-03-22t01
---

# refresh an ag db - 

## Description

refreshes an availability group database from a backup taken from a load server

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 <#
 
     .NOTES 
     Name: Availability Group Refresh
     Author: Rob Sewell http://sqldbawithabeard.com
     
     .DESCRIPTION 
         Refreshes an Availability group database from a backup taken from a Load Server
 
         YOU WILL NEED TO RESOLVE ORPHANED USERS IF REQUIRED
 #> 
 
 cls
 
     [System.Reflection.Assembly]::LoadWithPartialName(�Microsoft.SqlServer.SMO�)  | out-null
     [System.Reflection.Assembly]::LoadWithPartialName(�Microsoft.SqlServer.SMOExtended�)  | out-null
 
 
 $Date = Get-Date -Format ddMMyy
 
 
 $MyAgPrimaryPath = "SQLSERVER:\SQL\$PrimaryServer\DEFAULT\AvailabilityGroups\$AGName"
 $MyAgSecondaryPath = "SQLSERVER:\SQL\$SecondaryServer\DEFAULT\AvailabilityGroups\$AGName"
 $MyAgTertiaryPath = "SQLSERVER:\SQL\$TertiaryServer\DEFAULT\AvailabilityGroups\$AGName"
 
 $StartDate = Get-Date
 Write-Host "
 ##########################################
 Results of Script to refresh $DBName on
 $PrimaryServer , $SecondaryServer , $TertiaryServer
 on AG $AGName
 Time Script Started $StartDate
 
 " -ForegroundColor Green
 
 
 cd c:
 
 If(Test-Path $LoadDatabaseBackupFile){Remove-Item -Path $LoadDatabaseBackupFile -Force}
 If(Test-Path $DatabaseBackupFile){Remove-Item -Path $DatabaseBackupFile}
 If(Test-Path $LogBackupFile ) {Remove-Item -Path $LogBackupFile }
 
 Write-Host "Backup Files removed" -ForegroundColor Green
 
 Remove-SqlAvailabilityDatabase -Path SQLSERVER:\SQL\$SecondaryServer\DEFAULT\AvailabilityGroups\$AGName\AvailabilityDatabases\$DBName 
 
 Write-Host "Secondary Removed from Availability Group" -ForegroundColor Green
 
 Remove-SqlAvailabilityDatabase -Path SQLSERVER:\SQL\$TertiaryServer\DEFAULT\AvailabilityGroups\$AGName\AvailabilityDatabases\$DBName
 
 Write-Host "Tertiary removed from Availability Group" -ForegroundColor Green
 
 Remove-SqlAvailabilityDatabase -Path SQLSERVER:\SQL\$PrimaryServer\DEFAULT\AvailabilityGroups\$AGName\AvailabilityDatabases\$DBName
 
 Write-Host "Primary removed from Availability Group" -ForegroundColor Green
 
 Backup-SqlDatabase -Database $DBName -BackupFile $LoadDatabaseBackupFile -ServerInstance $LoadServer
 
 Write-Host "Load Database Backed up" -ForegroundColor Green
 
 $srv = New-Object Microsoft.SqlServer.Management.Smo.Server $PrimaryServer
 $srv.KillAllProcesses($dbname)
 
 Restore-SqlDatabase -Database $DBName -BackupFile $LoadDatabaseBackupFile  -ServerInstance $PrimaryServer -ReplaceDatabase
 
 Write-Host "Primary Database Restored" -ForegroundColor Green
 
 Backup-SqlDatabase -Database $DBName -BackupFile $LogBackupFile -ServerInstance $PrimaryServer -BackupAction 'Log'
 
 Write-Host "Primary Database Backed Up" -ForegroundColor Green
 
 $srv = New-Object Microsoft.SqlServer.Management.Smo.Server $SecondaryServer
 $srv.KillAllProcesses($dbname)
 
 Restore-SqlDatabase -Database $DBName -BackupFile $LoadDatabaseBackupFile-ServerInstance $SecondaryServer -NoRecovery -ReplaceDatabase 
 Restore-SqlDatabase -Database $DBName -BackupFile $LogBackupFile -ServerInstance $SecondaryServer -RestoreAction 'Log' -NoRecovery  -ReplaceDatabase
 
 Write-Host "Secondary Database Restored" -ForegroundColor Green
 
 $srv = New-Object Microsoft.SqlServer.Management.Smo.Server $TertiaryServer
 $srv.KillAllProcesses($dbname)
 
 Restore-SqlDatabase -Database $DBName -BackupFile $LoadDatabaseBackupFile -ServerInstance $TertiaryServer -NoRecovery -ReplaceDatabase
 Restore-SqlDatabase -Database $DBName -BackupFile $LogBackupFile -ServerInstance $TertiaryServer -RestoreAction 'Log' -NoRecovery  -ReplaceDatabase
 
 Write-Host "Tertiary Database Restored" -ForegroundColor Green
 
 Add-SqlAvailabilityDatabase -Path $MyAgPrimaryPath -Database $DBName 
 Add-SqlAvailabilityDatabase -Path $MyAgSecondaryPath -Database $DBName 
 Add-SqlAvailabilityDatabase -Path $MyAgTertiaryPath -Database $DBName 
 
 Write-Host "Database Added to Availability Group " -ForegroundColor Green
 
  $srv = New-Object Microsoft.SqlServer.Management.Smo.Server $PrimaryServer
     $AG = $srv.AvailabilityGroups[$AGName]
     $AG.DatabaseReplicaStates|ft -AutoSize
 
     $EndDate = Get-Date
     $Time = $EndDate - $StartDate
 Write-Host "
 ##########################################
 Results of Script to refresh $DBName on
 $PrimaryServer , $SecondaryServer , $TertiaryServer
 on AG $AGName
 Time Script ended at $EndDate and took
 $Time
 
 " -ForegroundColor Green
`

