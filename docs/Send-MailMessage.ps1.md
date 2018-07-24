---
Author: xxxxxx
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 4663
Published Date: 2016-12-04t17
Archived Date: 2016-03-19t00
---

# send-mailmessage.ps1 - 

## Description

from windows powershell cookbook (o’reilly) by lee holmes

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 ##############################################################################
 ##
 ##
 ##
 ##
 ##
 ##
 ##############################################################################
 
 param(
     [string[]] $To = $(throw "Please specify the destination mail address"),
 
     [string] $Subject = "<No Subject>",
 
     [string] $Body = $(throw "Please specify the message content"),
 
     [string] $SmtpHost = $(throw "Please specify a mail server."),
 
     [string] $From = "$($env:UserName)@example.com"
 )
 
 $email = New-Object System.Net.Mail.MailMessage
 
 foreach($mailTo in $to)
 {
     $email.To.Add($mailTo)
 }
 
 $email.From = $from
 $email.Subject = $subject
 $email.Body = $body
 
 $client = New-Object System.Net.Mail.SmtpClient $smtpHost
 $client.UseDefaultCredentials = $true
 $client.Send($email)
`

