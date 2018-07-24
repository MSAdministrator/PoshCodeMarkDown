---
Author: dbawithabeard
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5571
Published Date: 2016-11-04t19
Archived Date: 2016-11-06t01
---

# disk check &amp; email &amp; log - 

## Description

<#

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 <#
 .Synopsis
    Disk Check and email alert for Servers
 .DESCRIPTION
     Author - Rob Sewell
     Date - 29/10/2014
 
    This script will iterate through the list of servers in the $Servers variable and check the disk space
    for every disk.
 
    There are three warning levels currently set at 15%,5% and 1%
 
    If the free disk space for any disk is below a warning level, the script will check for
    the existence of a unique to the disk named text file located in the path of the $location 
    variable, presuming one does not exist, the script will create a text file and then send email
 
    If the script finds a text file it will not email
 
    Once the free space increases above the warning level the script will remove the text file
 
 .EXAMPLE
     The script is designed to run under a scheduled agent job on the SQL Server and will run every 
     5 minutes
     
 .NOTES
 
    The script will log to the file located at $LogFile location which will be deleted
 
 #>
 $Servers = Get-Content 'PATH\TO\Servers.txt'
  $WarningLevel = '15'
 $ErrorLevel = '5'
 $SevereLevel = '1'
 $Date = Get-Date
 $Logdate = Get-Date -Format yyyyMMdd
 $LogFile = $Location + 'logfile' + $LogDate+ '.txt'
 
 if(!(Test-Path $LogFile)) 
  {
 New-Item $Logfile -ItemType File
 }
 
 Get-ChildItem -Path $Location *logfile* |Where-Object {$_.LastWriteTime -gt (Get-Date).AddDays(7) }|Remove-Item -Force 
 
 
 foreach($Server in $Servers)
 {
 $Disks = Get-WmiObject win32_logicaldisk -ComputerName $Server | Where-Object {$_.drivetype -eq 3} -ErrorAction SilentlyContinue
 
 foreach($Disk in $Disks)
 {
 $ServerName = $Disk.__SERVER
 $VolumeName = $Disk.VolumeName
 $DriveLetter = $Disk.DeviceID.Chars(0)
 
 $UsedSpace = $TotalSpace - $FreeSpace
 
 $CheckFile = $Location + $Server + '_' + $DriveLetter + '_Warning.txt' 
 $CheckFileError = $Location + $Server + '_' + $DriveLetter + '_Error.txt'
 $CheckFileSevere = $Location + $Server + '_' + $DriveLetter + '_Severe.txt'
 
 if ($PercentFree -le $SevereLevel) 
  {
 
 if(Test-Path $CheckFileSevere)
 {
 $logentrydate = (Get-Date).DateTime
 $Log = $logentrydate + ' ' + $ServerName + ' ' +  $DriveLetter + ' ' +  $VolumeName + ' ' + $PercentFree +' -- ' + $CheckFileSevere + ' File Already Created - No Action Taken'
 Add-Content -Value $Log -Path $Logfile
 } 
 else 
 {
 
 New-Item $CheckFileSevere -ItemType File
 $logentrydate = (Get-Date).DateTime
 $Log = $logentrydate + ' ' + $ServerName + ' ' +  $DriveLetter + ' ' +  $VolumeName + ' ' + $PercentFree +' -- File Created'
 Add-Content -Value $Log -Path $Logfile
 $EmailBody = ''
 $EmailBody += "<html> <head>  <title> $Date DiskSpace Report</title>"
 $EmailBody += "<table width='100%'><tbody>"
 
 $EmailBody += "<tr>"
 $EmailBody += "</tr>"
 
 $EmailBody += "</tr>"
 $EmailBody += "<td width='15%' align='center'>Drive</td>"
 $EmailBody += "<td width='25%' align='center'>Drive Label</td>"
 $EmailBody += "<td width='15%' align='center'>Total Capacity(GB)</td>"
 $EmailBody += "<td width='15%' align='center'>Used Capacity(GB)</td>"
 $EmailBody += "<td width='15%' align='center'>Free Space(GB)</td>"
 $EmailBody += "<td width='15%' align='center'>Freespace %</td>"
 $EmailBody += "</tr>"
 
 $EmailBody += "<tr>"
 $EmailBody += "</tr>"
 
 $smtpServer = ""
 $To = ""
 $From = ""
 $Sender = ""
 $Subject = "URGENT Disk Space Alert 1%"
 $Body = $EmailBody
 $msg = new-object Net.Mail.MailMessage
 $smtp = new-object Net.Mail.SmtpClient($smtpServer)
 $smtp.port = '25'
 $msg.From = $From
 $msg.Sender = $Sender
 $msg.To.Add($To)
 $msg.Subject = $Subject
 $msg.Body = $Body
 $msg.IsBodyHtml = $True
 $smtp.Send($msg)
 $logentrydate = (Get-Date).DateTime
 $Log = $logentrydate + ' ' + $ServerName + ' ' +  $DriveLetter + ' ' +  $VolumeName + ' ' + $PercentFree +' -- Severe Email Sent'
 Add-Content -Value $Log -Path $Logfile
 }
 
 }
 
 elseif ($PercentFree -le $ErrorLevel) 
  {
 
 if(Test-Path $CheckFileError)
 {
 $logentrydate = (Get-Date).DateTime
 $Log = $logentrydate + ' ' + $ServerName + ' ' +  $DriveLetter + ' ' +  $VolumeName + ' ' + $PercentFree +' --' + $CheckFileError + ' File Already Created - No Action Taken'
 Add-Content -Value $Log -Path $Logfile
 } 
 
 else 
 {
 New-Item $CheckFileError -ItemType File
 $logentrydate = (Get-Date).DateTime
 $Log = $logentrydate + ' ' + $ServerName + ' ' +  $DriveLetter + ' ' +  $VolumeName + ' ' + $PercentFree +' -- File Created'
 Add-Content -Value $Log -Path $Logfile
 
 $EmailBody = ''
 $EmailBody += "<html> <head>  <title> $Date DiskSpace Report</title>"
 $EmailBody += "<table width='100%'><tbody>"
 
 $EmailBody += "<tr>"
 $EmailBody += "</tr>"
 
 $EmailBody += "</tr>"
 $EmailBody += "<td width='15%' align='center'>Drive</td>"
 $EmailBody += "<td width='25%' align='center'>Drive Label</td>"
 $EmailBody += "<td width='15%' align='center'>Total Capacity(GB)</td>"
 $EmailBody += "<td width='15%' align='center'>Used Capacity(GB)</td>"
 $EmailBody += "<td width='15%' align='center'>Free Space(GB)</td>"
 $EmailBody += "<td width='15%' align='center'>Freespace %</td>"
 $EmailBody += "</tr>"
 
 
 $EmailBody += "<tr>"
 $EmailBody += "<td align=center>$DriveLetter</td>"
 $EmailBody += "<td align=center>$VolumeName</td>"
 $EmailBody += "<td align=center>$TotalSpace</td>"
 $EmailBody += "<td align=center>$UsedSpace</td>"
 $EmailBody += "<td align=center>$FreeSpace</td>"
 $EmailBody += "</tr>"
 
 $smtpServer = ""
 $To = ""
 $From = ""
 $Sender = ""
 $Subject = " Disk Space Alert 5%"
 $Body = $EmailBody
 $msg = new-object Net.Mail.MailMessage
 $smtp = new-object Net.Mail.SmtpClient($smtpServer)
 $smtp.port = '25'
 $msg.From = $From
 $msg.Sender = $Sender
 $msg.To.Add($To)
 $msg.Subject = $Subject
 $msg.Body = $Body
 $msg.IsBodyHtml = $True
 $smtp.Send($msg)
 $logentrydate = (Get-Date).DateTime
 $Log = $logentrydate + ' ' + $ServerName + ' ' +  $DriveLetter + ' ' +  $VolumeName + ' ' + $PercentFree + ' -- Error Email Sent'
 Add-Content -Value $Log -Path $Logfile
 }
 
 
 
 }
 elseif ($PercentFree -le $WarningLevel) 
  {
 
 if(Test-Path $CheckFile)
 {
 $logentrydate = (Get-Date).DateTime
 $Log = $logentrydate + ' ' + $ServerName + ' ' +  $DriveLetter + ' ' +  $VolumeName + ' ' + $PercentFree + ' --' + $CheckFile + ' File Already Created - No Action Taken'
 Add-Content -Value $Log -Path $Logfile
 }
 
 else 
 {
 New-Item $CheckFile -ItemType File
 $logentrydate = (Get-Date).DateTime
 $Log = $logentrydate + ' ' + $ServerName + ' ' +  $DriveLetter + ' ' +  $VolumeName + ' ' + $PercentFree +' -- File Created'
 Add-Content -Value $Log -Path $Logfile
 $EmailBody = ''
 $EmailBody += "<html> <head>  <title> $Date DiskSpace Report</title>"
 $EmailBody += "<table width='100%'><tbody>"
 
 $EmailBody += "<tr>"
 $EmailBody += "</tr>"
 
 $EmailBody += "</tr>"
 $EmailBody += "<td width='15%' align='center'>Drive</td>"
 $EmailBody += "<td width='25%' align='center'>Drive Label</td>"
 $EmailBody += "<td width='15%' align='center'>Total Capacity(GB)</td>"
 $EmailBody += "<td width='15%' align='center'>Used Capacity(GB)</td>"
 $EmailBody += "<td width='15%' align='center'>Free Space(GB)</td>"
 $EmailBody += "<td width='15%' align='center'>Freespace %</td>"
 $EmailBody += "</tr>"
 
 
 $EmailBody += "<tr>"
 $EmailBody += "<td align=center>$DriveLetter</td>"
 $EmailBody += "<td align=center>$VolumeName</td>"
 $EmailBody += "<td align=center>$TotalSpace</td>"
 $EmailBody += "<td align=center>$UsedSpace</td>"
 $EmailBody += "<td align=center>$FreeSpace</td>"
 $EmailBody += "</tr>"
 
 $smtpServer = ""
 $To = ""
 $From = ""
 $Sender = ""
 $Subject = "Disk Space Alert"
 $Body = $EmailBody
 $msg = new-object Net.Mail.MailMessage
 $smtp = new-object Net.Mail.SmtpClient($smtpServer)
 $smtp.port = '25'
 $msg.From = $From
 $msg.Sender = $Sender
 $msg.To.Add($To)
 $msg.Subject = $Subject
 $msg.Body = $Body
 $msg.IsBodyHtml = $True
 $smtp.Send($msg)
 $logentrydate = (Get-Date).DateTime
 $Log = $logentrydate + ' ' + $ServerName + ' ' +  $DriveLetter + ' ' +  $VolumeName + ' ' + $PercentFree + ' -- Warning Email Sent'
 Add-Content -Value $Log -Path $Logfile
 }
 
 
 
 }
 else 
  {
 if(Test-Path $CheckFileError)
 {
 Remove-Item $CheckFileError -Force
   $logentrydate = (Get-Date).DateTime
 $Log = $logentrydate + ' ' + $ServerName + ' ' +  $DriveLetter + ' ' +  $VolumeName + ' ' + $PercentFree + ' -- File Removed'
 Add-Content -Value $Log -Path $Logfile
 }
   if(Test-Path $CheckFile)
 {
 Remove-Item $CheckFile -Force
   $logentrydate = (Get-Date).DateTime
 $Log = $logentrydate + ' ' + $ServerName + ' ' +  $DriveLetter + ' ' +  $VolumeName + ' ' + $PercentFree +' --  File Removed'
 Add-Content -Value $Log -Path $Logfile
 }
 if(Test-Path $CheckFileSevere)
 {
 Remove-Item $CheckFileSevere -Force
   $logentrydate = (Get-Date).DateTime
 $Log = $logentrydate + ' ' + $ServerName + ' ' +  $DriveLetter + ' ' +  $VolumeName + ' ' + $PercentFree + ' -- File Removed'
 Add-Content -Value $Log -Path $Logfile
 }
 
 $logentrydate = (Get-Date).DateTime
 $Log = $logentrydate + ' ' + $ServerName + ' ' +  $DriveLetter + ' ' +  $VolumeName + ' ' + $PercentFree + ' -- Checked No Action Taken'
 Add-Content -Value $Log -Path $Logfile
 }
 
 }
 }
`

