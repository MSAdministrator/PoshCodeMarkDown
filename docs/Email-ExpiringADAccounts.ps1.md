---
Author: andrewjh
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 3153
Published Date: 2012-01-07t13
Archived Date: 2012-01-22t04
---

# email-expiringadaccounts - 

## Description

quick and simple, send an email to ad accounts expiring before a specified date.  the $body is specific to my org needs, but simply customize this to suit.  give values to $smtp,$from,$subject etc and away you go.

## Comments



## Usage



## TODO



## function

``

## Code

`#
 #
 Function GetMsgBody {
 	Write-Output @"
 		<p>Dear $name,</p>
 		Your ABC network account is about to expire.<br/>
 		Please email helpdesk@abc.com or simply hit 'reply', and include the following details to have your account extended.<br/>
 		<br/>
 		Your name:<br/>
 		The department you currently volunteer in:<br/>
 		The staff supervisor's name you currently report to:<br/>
 		The current status of your involvement with ABC (Staff/Student/Volunteer):<br/>
 		<br/>
 		Kind Regards,<br/>
 		ABC IT Services
 "@
 }
 
 Get-QADUser -AccountExpiresBefore "31/01/2012" -Enabled -SizeLimit 0 | ForEach-Object {
 	$email=		$_.mail
 	$name=		$_.givenName
 	[string]$body=	GetMsgBody
 	
 	Send-MailMessage -BodyAsHtml:$true -Body $body -To $email -From $from -SmtpServer $smtp -Subject $subject
 	}
`

