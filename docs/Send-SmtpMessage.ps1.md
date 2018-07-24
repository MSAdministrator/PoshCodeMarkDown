---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 1.2
Encoding: ascii
License: cc0
PoshCode ID: 5990
Published Date: 2016-08-26t09
Archived Date: 2016-05-17t11
---

# send-smtpmessage - 

## Description

send an email via the gmail smtp server (or any server, really, but there’s some extra code in here and defaults that will make it work with gmail so you won’t have to do research).  note

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 ###################################################################################################
 ###################################################################################################
 ###################################################################################################
 param (
    [Net.Mail.MailAddress]$to = $(Read-Host "Email address to send to (eg: receiver@hotmail.com)")
   ,[string]$Subject = $(Read-Host "Email Subject")
   ,[string]$Body = $(Read-Host "Main Email Body")
   ,[Net.NetworkCredential]$credentials = $(Get-Credential)
 
   ,[Net.Mail.MailAddress]$from = $null
 )
 $credentials.UserName += "@" + $credentials.Domain
 $credentials.Domain = ""
 
 if(!$from) { 
   $from = $credentials.UserName
 }
 
 if($Port) {
    $mail = new-object Net.Mail.SmtpClient $Server,$Port
 } else {
    $mail = new-object Net.Mail.SmtpClient $Server
 }
 
 $mail.Credentials = $credentials
 $mail.EnableSsl = $true
 
 $mail.Send($From,$To,$Subject,$Body)
`

