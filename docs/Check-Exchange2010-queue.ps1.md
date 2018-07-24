---
Author: paperclips (the dark lord)
Publisher: 
Copyright: 
Email: magiconion_m@hotmail.com
Version: 0.1
Encoding: utf-8
License: cc0
PoshCode ID: 2674
Published Date: 2011-05-13t01
Archived Date: 2011-05-16t21
---

# check exchange2010 queue - 

## Description

purpose

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 
 $s = New-PSSession -ConfigurationName Microsoft.Exchange -ConnectionUri http://ukvms-wcas2/PowerShell/ -Authentication Kerberos
 Import-PSSession $s
 
 Add-PSSnapin Microsoft.Exchange.Management.PowerShell.E2010
 . $env:ExchangeInstallPath\bin\RemoteExchange.ps1
 Connect-ExchangeServer -auto
 
 $filename = �c:\temp\qu.txt�
 Start-Sleep -s 10
 if (Get-ExchangeServer | Where { $_.isHubTransportServer -eq $true } | get-queue | Where-Object { $_.MessageCount -gt 10 })
 
 {
 
 Get-ExchangeServer | Where { $_.isHubTransportServer -eq $true } | get-queue | Where-Object { $_.MessageCount -gt 10 } | Format-Table -Wrap -AutoSize | out-file -filepath c:\temp\qu.txt
 Start-Sleep -s 10
 
 $smtpServer = �xxx.xxx.xxx.xxx�
 $msg = new-object Net.Mail.MailMessage
 $att = new-object Net.Mail.Attachment($filename)
 $smtp = new-object Net.Mail.SmtpClient($smtpServer)
 $msg.From = �Monitor@contoso.com�
 $msg.To.Add("admin1@mycompany.com")
 #$msg.To.Add("admin2@mycompany.com")
 #$msg.To.Add("admin3@mycompany.com")
 #$msg.To.Add("admin4@mycompany.com")
 $msg.Subject = �CAS SERVER QUEUE THRESHOLD REACHED - PLEASE CHECK EXCHANGE QUEUES�
 $msg.Body = �Please see attached queue log file for queue information�
 $msg.Attachments.Add($att)
 $smtp.Send($msg)
 
 }
`

