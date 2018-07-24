---
Author: rob sewell http
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: utf-8
License: cc0
PoshCode ID: 4463
Published Date: 2016-09-11t19
Archived Date: 2016-04-06t05
---

# show last backup server - show-lastserverbackup.ps1

## Description

#############################################################################################

## Comments



## Usage



## TODO



## function

`show-lastserverbackup`

## Code

`#
 #
 
   #############################################################################################
 #
 #
 
 Function Show-LastServerBackup ($SQLServer)
 {
 
 $server = new-object "Microsoft.SqlServer.Management.Smo.Server" $SQLServer
 
 $Results = @();
 
 foreach($db in $server.Databases)
 {
 $DBName = $db.name
 $LastFull = $db.lastbackupdate
 if($lastfull -eq '01 January 0001 00:00:00')
 {$LastFull = 'NEVER'}
 
 $LastDiff = $db.LastDifferentialBackupDate  
 if($lastdiff -eq '01 January 0001 00:00:00')
 {$Lastdiff = 'NEVER'}
                                                                                                                                                         
 $lastLog = $db.LastLogBackupDate 
 if($lastlog -eq '01 January 0001 00:00:00')
 {$Lastlog= 'NEVER'}
 
         $TempResults = New-Object PSObject;
         $TempResults | Add-Member -MemberType NoteProperty -Name "Server" -Value $Server;
         $TempResults | Add-Member -MemberType NoteProperty -Name "Database" -Value $DBName;
         $TempResults | Add-Member -MemberType NoteProperty -Name "Last Full Backup" -Value $LastFull;
         $TempResults | Add-Member -MemberType NoteProperty -Name "Last Diff Backup" -Value $LastDiff;
         $TempResults | Add-Member -MemberType NoteProperty -Name "Last Log Backup" -Value $LastLog;
         $Results += $TempResults;
 
 
 }
 $Results|Format-Table -auto
 }
`

