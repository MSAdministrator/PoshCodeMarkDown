---
Author: rob sewell http
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: utf-8
License: cc0
PoshCode ID: 4462
Published Date: 2013-09-11t18
Archived Date: 2013-09-17t23
---

# show last db backup - show-lastdatabasebackup.ps1

## Description

#############################################################################################

## Comments



## Usage



## TODO



## function

`show-lastdatabasebackup`

## Code

`#
 #
 
   #############################################################################################
 #
 #
 
 Function Show-LastDatabaseBackup ($SQLServer, $database)
 {
 
 $server = new-object "Microsoft.SqlServer.Management.Smo.Server" $SQLServer
 $db = $server.Databases[$database]
 
 Write-Host "Last Full Backup"
 $LastFull = $db.LastBackupDate
 if($lastfull -eq '01 January 0001 00:00:00')
 {$LastFull = 'NEVER'}
 Write-Host $LastFull
 
 Write-Host "Last Diff Backup"
 $LastDiff = $db.LastDifferentialBackupDate  
 if($lastdiff -eq '01 January 0001 00:00:00')
 {$Lastdiff = 'NEVER'}
 Write-Host $Lastdiff
 
 Write-Host "Last Log Backup"                                                                                                                                                         
 $lastLog = $db.LastLogBackupDate 
 if($lastlog -eq '01 January 0001 00:00:00')
 {$Lastlog= 'NEVER'}
 Write-Host $lastlog
 
 }
`

