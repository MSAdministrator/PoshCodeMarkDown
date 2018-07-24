---
Author: victor flores
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 3076
Published Date: 2011-12-01t12
Archived Date: 2011-12-04t06
---

# sybase error log check - 

## Description

description

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 
 
 
 #
 
 
 
 
 
 
 del G:\YourScriptDirectory\ErrorsFound.log
 del G:\YourScriptDirectory\errlogfile.txt
 
 
 
 #
 function EmailResults 
 {
 $mailmessage = New-Object system.net.mail.mailmessage 
 $mailmessage.from       = ("YourDBASendingAccount@YourCompanyEmailServer,com") 
 $mailmessage.To.add("DBA1@your.company.com,DBA2@your.company.com")
 $mailmessage.Subject     = $Subject
 $mailmessage.Body        = $EmailBody
 $mailmessage.IsBodyHTML  = $true
 $SMTPClient             = New-Object Net.Mail.SmtpClient("YourCompanySMTPServer.com", 25) 
 $SMTPClient.Credentials = New-Object System.Net.NetworkCredential("YourDBASendingAccount@YourCompanyEmailServer.com", "password")
 $SMTPClient.Send($mailmessage)
 }
 
 
 #####$today = Get-Date
 #####$daysback = New-Timespan -days 17
 #####$cutoff = $today - $daysback
 
 
 
 $hoursback = New-Timespan -hours 12
 $cutoff = $today - $hoursback
 
 
 
 
 
  
 $LogFileByDate = Get-Content E:\sybase\ASE-15_0\install\cbstest.log | Foreach-Object { $elements = $_.Split("`t");`
 $rv = 1 | Select-Object date, message;$rv.date = if($elements[0] -notmatch "^\d\d:\d{5}:\d{5}:\d{4}/\d\d/\d\d\s\d\d:\d\d:\d\d\.\d\d"){$elements >> G:\YourScriptDirectory\errlogfile.txt;"UNKNOWN"}else{ [DateTime]($elements[0].SubString(15,10))}`
 #$rv.Date = [DateTime]($elements[0].SubString(15,10));`
 $rv.Message = $elements[1]; $elements } | Where-Object { $rv.Date -gt $cutoff } 
 
 
 
 $LogFileByDate| out-file G:\YourScriptDirectory\LogFileByDate.txt
 
 
 $ErrorsFound= Select-String -Path G:\YourScriptDirectory\LogFileByDate.txt  -pattern "error","warning","severity",`
               "fail","full", "couldn","not found","not valid","invalid", "threshold","unmirror",`
               "mirror","deadlock", "allow","NO_LOG","logsegment","syslogs"  
 
 
 
 
 
 $ErrorsFound | out-file G:\YourScriptDirectory\ErrorsFound.log
 
 
 
 
 $EmailBody = (gc G:\YourScriptDirectory\ErrorsFound.log | out-string)
 $Subject = 'Sybase Error Log Report for YourSybaseServerName - Sybase Errors Have Been Encountered on YourSybaseServerName'
 
 
 if ($emailbody.Length -gt 0)
 {
 EmailResults $Subject $Body
 }
`

