---
Author: dan dill
Publisher: 
Copyright: 
Email: 
Version: 1.0
Encoding: ascii
License: cc0
PoshCode ID: 3760
Published Date: 2012-11-12t06
Archived Date: 2012-11-16t06
---

# audit iphone/palm users - 

## Description

this script is intended to use iis logs to audit owa/activesync logs for syncing of mail from an iphone or a palm device. this script is not perfect, nor the prettiest thing in the world but it works.  it could be further added to parse for windows mobile devices.  if it was really slick it would grab all the unique values in the devicetype= portion and then automatically include all mobile types.  you can email the results to yourself in $to varible.

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 $Daysold = 1
 $Date = (get-date).adddays(-$daysold)
 $servers = "server1", "server2", "server3"
 foreach ($s in $servers) 
     {
     Write-host -ForegroundColor Blue "Checking server $s for files from the last $daysold day(s)"
     $logfiles += gci -path \\$s\c$\inetpub\logs\LogFiles\W3SVC1 | where {$_.LastWriteTime -gt $date}
     }
     
 Foreach ($l in $logfiles)
     {
     
     Write-host "Processing "$l.fullname
     Copy-item $l.fullname -Destination $pwd.path
 	$palmusers +=  gc $l.name | where {$_ -match "DeviceType=Palm"}
 	$iphoneusers +=  gc $l.name | where {$_ -match "DeviceType=iPhone"}
     Remove-Item $l.name
     }
 $iuser = @()
 $puser = @()
 foreach ($l in $iphoneusers | where {$_ -ne $null})
     {
     $u = $l.split(" ")[7]
     if ($iuser -notcontains $u)
         {
         $iuser += "$u"
         }
     $u = $null
     }
 	foreach ($l in $palmusers | where {$_ -ne $null})
     {
     $u = $l.split(" ")[7]
     if ($puser -notcontains $u)
         {
         $puser += "$u"
         }
     $u = $null
     }
 $body = "<!DOCTYPE html PUBLIC `"-//W3C//DTD XHTML 1.0 Strict//EN`"  `"http://www.w3.org/TR/xhtml1/DTD/xhtml1-strict.dtd`">"
 $body += "<html xmlns=`"http://www.w3.org/1999/xhtml`">"
 $body += "<head>"
 $body += "<title>iPhone Users</title>"
 $body += "</head><body>"
 $body += "<table border=1>"
 $body += "<colgroup>"
 $body += "<col/>"
 $body += "</colgroup>"
 $body += "<tr><td><b>iPhone Users</b></td></tr>"
 foreach ($y in $iuser)
     {
     $body += "<tr><td>$y</td></tr>"
     }
 $body += "<tr><td></td></tr>"
 $body += "<br>"
 $body += "<tr><td><b>Palm Users</b></td></tr>"
 foreach ($y in $puser)
     {
     $body += "<tr><td>$y</td></tr>"
     }
 $body += "</table>"
 $body += "<br>Audited servers:  $servers <br>"
 $body += "Audited for:  DeviceType=Palm and DeviceType=iPhone"
 $body += "</body></html>"
 
 $smtpServer = "yourmailserver"
 $mailer = new-object Net.Mail.SMTPclient($smtpserver)	
 $From = "dontreplyiamascript@domain.com"
 $To = "youremail@yourdomain.com"
 $subject = "Mobile users syncing through OWA in the last $daysold day(s)"
 $msg = new-object Net.Mail.MailMessage($from,$to,$subject,$body)	
 $msg.IsBodyHTML = $true
 $mailer.send($msg)
 
 clear-variable logfiles
 clear-variable servers
 clear-variable daysold
`

