---
Author: dvsbobloblaw
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6471
Published Date: 2016-08-12t09
Archived Date: 2016-08-13t22
---

# werwerwer - 

## Description

http

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
 An attempt to send text messages to phones. 
 
 .DESCRIPTION
 Have you ever wrote a deamon script that notifies you when a server is down or something else. Wouldn't it be nice to get a TEXT (SMS) message.
 
 .EXAMPLE 
 Send-TextMessage -Port 587 -SmtpServer 'smtp.gmail.com' -UseSsl -Body 'Hello World' -From 'evilperson@gmail.com' -Subject '>=)' -Number '7238675309' -ServiceProvider Verizon -Credential (Get-Credential)
 
 This will send a text message to 7238675309 from evilperson@gmail.com
 
 .NOTES
 This is pretty lame if you already know the person's text email address gateway thing.
 #>
 
 
 [CmdletBinding(HelpUri='http://go.microsoft.com/fwlink/?LinkID=135256')]
 param(
 
     [Parameter(Mandatory=$true)]
     [Alias('Message')]
     [ValidateNotNullOrEmpty()]
     [ValidateLength(1,159)]
     [string]$Body,
 
     [Parameter(Mandatory=$true)]
     [ValidateNotNullOrEmpty()]
     [string]$From,
 
     [Parameter()]
     [Alias('ComputerName')]
     [ValidateNotNullOrEmpty()]
     [string]$SmtpServer,
 
     [Parameter(Mandatory=$true)]
     [Alias('sub')]
     [ValidateNotNullOrEmpty()]
     [string]$Subject,
 
     [Parameter(Mandatory=$true)]
     [Alias('To','PhoneNumber')]
     [ValidateNotNullOrEmpty()]
     [string]$Number,
 
     [ValidateNotNullOrEmpty()]
     [pscredential]$Credential,
 
     [switch]$UseSsl,
 
     [ValidateRange(0, 2147483647)]
     [int]$Port,
 
     [Parameter(Mandatory=$true)]
     [ValidateSet('USCellular','Verizon','ATT','Sprint','Virgin','T-Mobile','Cricket')]
     [string]$ServiceProvider
 )
 
 switch ($ServiceProvider) {
     'USCellular' {$Recipient="$Number@email.uscc.net"}
     'Verizon' {$Recipient="$Number@vtext.com"}
     'ATT' {$Recipient="$Number@vmobl.com"}
     'Sprint' {$Recipient="$Number@messaging.sprintpcs.com"}
     'Virgin' {$Recipient="$Number@vmobl.com"}
     'T-Mobile' {$Recipient="$Number@tmomail.net"}
     'Cricket' {$Recipient="$Number@sms.mycricket.com"}
     default {throw 'How did this happen?'}
 }
 
 $Information = @{Body=$Body;From=$From;SmtpServer=$SmtpServer;Subject=$Subject;To=$Recipient;Credential=$Credential;UseSsl=$UseSsl;Port=$Port}
 
 try{
     Send-MailMessage @Information
 }
 catch{
     throw "Awe, it didn't work."
 }
`

