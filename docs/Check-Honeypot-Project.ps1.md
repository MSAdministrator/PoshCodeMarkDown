---
Author: munsonisim
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5715
Published Date: 2015-01-26t20
Archived Date: 2015-01-28t09
---

# check honeypot project - 

## Description

this script will take a list of ip’s in an input csv (octet.csv) and check each ip at honeypotproject.org to see if the ip is listed.

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 $IPADDRESSES = Import-Csv '<Path to CSV>'
 $Smtpserver = '<Mail server Info>'
 $To = '<To Email Address>'
 
 foreach ($IPADD in $IPADDRESSES)
 {
 [string]$IP1 = $IPADD.Octet
 [string]$Outfilehtml = 'c:\'+$IP1+'.html'
 [string]$URL = "http://www.projecthoneypot.org/ip_"+$IP1
 [string]$positive = "The Project Honey Pot system has detected behavior from the IP address"
 [string]$negitive = "We don't have data on this IP currently. If you know something, you may"
 $web = Invoke-WebRequest -Uri $URL
 $string = $web.Content
 #$IP1
 if ($string -match $positive) {
     Send-MailMessage -SmtpServer $Smtpserver -To $To -From 'Honeypot_YES@yourdomain.com' -BodyAsHtml $string -Subject $IP1
 } 
 if ($string -match $negitive) {
 	Send-MailMessage -SmtpServer $Smtpserver -To $To -From 'Honeypot_NO@yourdomain.com' -Subject $IP1 
 }
 }
`

