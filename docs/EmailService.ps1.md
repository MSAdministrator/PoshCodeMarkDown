---
Author: johnny reel
Publisher: 
Copyright: 
Email: 
Version: 1.04
Encoding: ascii
License: cc0
PoshCode ID: 4652
Published Date: 2013-11-27t16
Archived Date: 2013-12-06t18
---

# emailservice - 

## Description

simple one liner that emails the status of a service(s) to a recipient.

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 
 
 
 Send-MailMessage -To "user@company.com" -From "sender@company.com" -SmtpServer <servername> -Subject "<service or server name> Service Status" -body ((gsv -cn <servrename> -Name <servicename>) | Out-string)
 
 exit 4
`

