---
Author: shilezi
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: utf-8
License: cc0
PoshCode ID: 3998
Published Date: 2013-03-05t14
Archived Date: 2013-03-10t01
---

# farm backup - 

## Description

site collection by site collection backup of sharepoint 2010 farm. script is as is and use at your discretion. i take no credit for the script, other than modifying it for my purposes. i took bits of scripts from 2 sites http

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 ##################################
 
 
 ##################################
 
 
 Add-PSSnapin Microsoft.SharePoint.Powershell -ErrorAction SilentlyContinue  
  
  try
  {
 
    
 $backupRoot = "\\SERVER\SPFarmBackUP\" 
 $logPath = Join-Path $backupRoot "_logs" 
 $tmpPath = Join-Path $backupRoot "_tmp" 
 $clearUpOldFiles = 0  
 $removeFilesOlderThanDays = -1 
 $todaysBackupFolder = Join-Path $backupRoot ((Get-Date).DayOfWeek.toString())  
  
 if (-not (Test-Path $logPath)) {   
   New-Item $logPath -type directory 
 }  
  
 if (-not (Test-Path $tmpPath)) {   
   New-Item $tmpPath -type directory 
 }  
  
 if (-not (Test-Path $todaysBackupFolder)) {   
   New-Item $todaysBackupFolder -type directory 
 }  
  
 Start-Transcript -Path (Join-Path $logPath ((Get-Date).ToString('MMddyyyy_hhmmss') + ".log"))
 
 $TXTFile = Join-Path $logPath ((Get-Date).ToString('MMddyyyy_hhmmss') + ".log")
 
 foreach ($webApplication in Get-SPWebApplication) {     
   Write-Host     
   Write-Host     
   Write-Host "Processing $webApplication"    
   Write-Host "*******************************"          
  
   foreach ($site in $webApplication.Sites) {                 
     $name = $site.Url.Replace("http://", "").Replace("https://", "").Replace("/", "_").Replace(".", "_");         
     [System.Text.RegularExpressions.Regex]::Replace($name,"[^1-9a-zA-Z_]","_");                  
     $backupPath = Join-Path $tmpPath ($name + (Get-Date).ToString('MMddyyyy_hhmmss') + ".bak")                  
  
     Write-Host "Backing up $site to $backupPath"         
     Write-Host                  
      
     Backup-SPSite -Identity $site.Url -Path $backupPath     
   }          
  
   Write-Host "*******************************"     
   Write-Host "*******************************" 
 }  
  
 Write-Host 
 Write-Host  
  
 if ($clearUpOldFiles -eq 1) {   
   Write-Host "Cleaning up the folder $todaysBackupFolder"   
   Remove-Item ($todaysBackupFolder + "\*")  
 }  
  
 Write-Host "Moving backups from $tmpPath to $todaysBackupFolder" 
 Move-Item -Path ($tmpPath + "\*") -Destination $todaysBackupFolder 
 if ($removeFilesOlderThanDays -gt 0) {   
   Write-Host "Checking removal policy on $todaysBackupFolder"   
   $toRemove = (Get-Date).AddDays(-$removeFilesOlderThanDays)   
   $filesToRemove = Get-ChildItem $todaysBackupFolder | Where {$_.LastWriteTime -le �$toRemove�}      
  
   foreach ($fileToRemove in $filesToRemove)  {     
     Write-Host "Removing the file $fileToRemove because it is older than $removeFilesOlderThanDays days"      
     Remove-Item (Join-Path $todaysBackupFolder $fileToRemove) | out-null   
   } 
 } 
   
 Stop-Transcript
 
     
   $today = (Get-Date -Format MM-dd-yyyy)
   $emailFrom = "admin@domain.com"
   $emailTo = "admin@domain.com"
   $subject = "Site collections on Farm Backup Report for this day "+"$today"
   $body = "The SharePoint backup of all Site collections in the farm has been run and was successful this day "+"$today"
   $smtpServer = "smtp.domain.com"
   $smtp = new-object Net.Mail.SmtpClient($smtpServer)
   Send-MailMessage -SmtpServer $smtpServer -From $emailFrom -To $emailTo -Subject $subject -Body $body -Attachment $TXTFile
   
 
 }
 
 catch
 {
   [System.Exception] 
   $ErrorMessage = $_.Exception.Message
   $emailFrom = "admin@domain.com"
   $emailTo = "admin@domain.com"
   $subject = "Site collections on Farm Backup Report for this day "+"$today"
   $body = "The SharePoint backup of all Site collections in the farm has been run and the Job failed today, the "+"$today because... $ErrorMessage."
   $smtpServer = "smtp.domain.com"
   $smtp = new-object Net.Mail.SmtpClient($smtpServer)
   Send-MailMessage -SmtpServer $smtpServer -From $emailFrom -To $emailTo -Subject $subject -Body $body
 }
`

