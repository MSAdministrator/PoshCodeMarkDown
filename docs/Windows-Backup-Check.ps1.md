---
Author: paulo seabra
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6519
Published Date: 2016-09-14t16
Archived Date: 2016-10-31t13
---

# windows backup check - 

## Description

powershell script to verify if windows backup has finished sucessfully or not. on error or warning it sends an email with the server name and the error/warning. on success it does nothing (means the backup finished successfully)

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 
 $Computer=hostname
 $Events=Get-EventLog -LogName Application -EntryType Warning,Error -Source Microsoft-Windows-Backup -After (Get-Date).AddHours(-24)
 
 $PSEmailServer = "fqdn of email server"
 $EmailFrom="youremail@yourdomain.xx"
 $EmailTo="youremail@yourdomain.xx"
 $EmailSubject="Backup - $Computer"
 
 
 $EmailBody="<html><head><meta http-equiv=`"Content-Type`" content=`"text/html; charset=utf-8`"></head>"+ $Computer + "<br><br>"
 If (!$Events) {exit}
 foreach ($Event in $Events){
 if ($Events)
 {    $Emailbody = $Emailbody + $Event.message + "<br>" +"<br>"}
 }
 
 Send-MailMessage -From $EmailFrom -To $EmailTo -Subject $EmailSubject -Body $EmailBody -BodyAsHtml -Encoding ([System.Text.Encoding]::UTF8)
`

