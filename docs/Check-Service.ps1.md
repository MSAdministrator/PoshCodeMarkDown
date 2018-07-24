---
Author: john kroes
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: utf-8
License: cc0
PoshCode ID: 1229
Published Date: 2012-07-23t15
Archived Date: 2012-04-03t18
---

# check service - 

## Description

use to check a service on a remote server and if it is not running, start it and send an email to warn. added a ping request to make sure the server is up before checking the service.

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 ####################################################################################
 ####################################################################################
 
 $erroractionpreference = "SilentlyContinue"
 
 
  $ping = new-object System.Net.NetworkInformation.Ping
     $rslt = $ping.send($i)
         if ($rslt.status �eq [System.Net.NetworkInformation.IPStatus]::Success)
 {
         $b = get-wmiobject win32_service -computername $i -Filter "Name = '$service'"
 
 	If ($b.state -eq "stopped")
 	{
 	$b.startservice()
 
 	$emailFrom = "services@yourdomain.com"
 	$emailTo = "you@yourdomain.com"
 	$subject = "$service Service has restarted on $i"
 	$body = "The $service service on $i has crashed and been restarted"
 	$smtpServer = "xx.yourdomain.com"
 	$smtp = new-object Net.Mail.SmtpClient($smtpServer)
 	$smtp.Send($emailFrom, $emailTo, $subject, $body)
 	}
 
 }
`

