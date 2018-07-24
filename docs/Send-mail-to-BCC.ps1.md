---
Author: thomas lee
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 811
Published Date: 2009-01-18t05
Archived Date: 2017-03-19t04
---

# send mail to bcc - 

## Description

this script is a re-developed msdn sample using powershell. it creates an email message then sends it with a bcc.

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 <#
 .SYNOPSIS
     Send mail to BCC using PowerShell
 .DESCRIPTION
     This script is a re-developed MSDN Sample using PowerShell. It creates
     an email message then sends it with a BCC.
 .NOTES
     File Name  : Send-BCCMail.ps1
 	Author     : Thomas Lee - tfl@psp.co.uk
 	Requires   : PowerShell V2 CTP3
 .LINK
     Original Sample Posted to
 	http://pshscripts.blogspot.com/2009/01/send-bccmailps1.html
 	MSDN Sample and details at:
 	http://msdn.microsoft.com/en-us/library/system.net.mail.mailaddresscollection.aspx
 .EXAMPLE
     PS C:\foo> .\Send-BCCMail.ps1
     Sending an e-mail message to The PowerShell Doctor and "Thomas Lee" <tfl@reskit.net>
 #>
 
 ###
 ###
 
 $From    = New-Object system.net.Mail.MailAddress "tfl@cookham.net", "Thomas Lee"
 $To      = new-object system.net.mail.mailaddress "doctordns@gmail.com", "The PowerShell Doctor"
 $Bcc     = New-Object system.Net.Mail.mailaddress "tfl@reskit.net", "Thomas Lee"
 $Message = New-Object system.Net.Mail.MailMessage $From, $To
 
 $Message.Subject = "Using the SmtpClient class and PowerShell."
 $Message.Body    = "Using this feature, you can send an e-mail message from an"
 $Message.Body   += "application very easily. `nEven better, you do it with PowerShell!"
 
 $Message.Bcc.Add($bcc);
 
 $Server = "localhost"
 $Client = New-Object System.Net.Mail.SmtpClient $server
 $Client.Credentials = [System.Net.CredentialCache]::DefaultNetworkCredentials
 "Sending an e-mail message to {0} and {1}" -f $to.DisplayName, $Message.Bcc.ToString()
 
 try {
     $client.Send($message);
 }  
 catch {
 "Exception caught in CreateBccTestMessage(): {0}" -f $Error[0]
 }
`

