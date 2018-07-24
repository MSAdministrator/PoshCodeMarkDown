---
Author: james day
Publisher: 
Copyright: 
Email: 
Version: 3.0
Encoding: ascii
License: cc0
PoshCode ID: 4587
Published Date: 2014-11-06t11
Archived Date: 2014-11-15t21
---

# windows server backup - 

## Description

windows server backup script. backs up to nas location, include facility for rotation and email notification

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 <#
 Author: James Day
 Created: 31st October 2013
 
 Original script source taken from http://gallery.technet.microsoft.com/scriptcenter/WSB-Backup-network-email-9793e315
 Full Credit goes to the Original author Augagneur Alexandre. Without your initial script I wouldn't have got this far.
 
 Tested on Server 2012 using Powershell version 3.0
 
 USE AT YOUR OWN RISK.
 #>
 Import-Module WindowsServerBackup
  
 #------------------------------------------------------------------ 
 #------------------------------------------------------------------  
  
 $Nas = "\\NAS" 
  
 $HomeBkpDir = ($Nas+"\BACKUP") 
  
 $Filename = Get-Date -Format yyyy-MM-dd_hhmmss
  
 $MaxBackup = 2
  
 $Volumes = Get-WBVolume -AllVolumes | Where-Object { $_.Property -notlike "Critical*" } 
   
 #------------------------------------------------------------------  
 #$MaxBackup (Not called if $MaxBackup equals 0) 
 #------------------------------------------------------------------  
 function Rotation() 
 {  
  $Backups = @(Get-ChildItem -Path $HomeBkpDir | Sort-Object -property Name) 
  
  $NbrBackups = $Backups.count 
  
  $i = 0 
   
  while ($NbrBackups -ge $MaxBackup) 
  { 
   $($Backups[$i].fullname) | Remove-Item -Recurse
   $NbrBackups -= 1 
   $i++ 
  } 
 } 
   
 #------------------------------------------------------------------ 
 #------------------------------------------------------------------  
 function EmailNotification() 
 { 
  Start-Sleep -Seconds 120
  $from = "backup@example.com" 
  
  $to = "postmaster@example.com" 
  
  $smtpserver = "server" 
   
  $Subject = $env:computername+": Backup report of "+(Get-Date) 
   
  $report = get-wbjob -previous 1 
  $success = $report.SuccessLogPath
  $Failure = $report.FailureLogPath
  $body = "Success and Failure logs are attached" 
 
  Send-MailMessage -to $to -from $from  -subject $Subject -body $body -attachments $success,$failure -smtpserver $smtpserver
 } 
  
 #------------------------------------------------------------------ 
 #------------------------------------------------------------------  
  
 if ($MaxBackup -ne 0) 
 { 
  Rotation 
 } 
   
 New-Item ($HomeBkpDir+"\"+$Filename)  -Type Directory | Out-Null 
   
 $WBPolicy = New-WBPolicy 
   
 Add-WBBareMetalRecovery -Policy $WBPolicy | Out-Null 
   
 $BackupLocation = New-WBBackupTarget -network ($HomeBkpDir+"\"+$Filename) 
 Add-WBBackupTarget -Policy $WBPolicy -Target $BackupLocation -force | Out-Null 
   
 if ($Volumes -ne $null) 
 { 
  Add-WBVolume -Policy $WBPolicy -Volume $Volumes | Out-null 
 } 
 Set-WBVssBackupOptions -policy $WBPolicy -vssfullbackup | Out-null
 
 $WBPolicy
 
 Start-WBBackup -Policy $WBPolicy 
   
 EmailNotification
`

