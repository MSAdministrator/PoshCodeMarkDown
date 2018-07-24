---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 631
Published Date: 
Archived Date: 2008-10-10t21
---

# citrix license info - 

## Description

this is a script i wrote to pull information from the citrix license web console by dumpingthe contents and searching for the area needed.  a better way of doing this is by using the bsonposh wmi script but unfortunatly there is an issue with the wmi classes on some installations as described on my blog here….http

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 ###########################################
 ###########################################
 
 param( [string] $sendmailsched )
 
 Function Sendemail ($LicTypeText, $InstalledLicNum, $InUseNum, $PercentageNum)
 {
 $smtpServer = "mysmtpserver.co.uk"
 
 $msg = new-object Net.Mail.MailMessage
 $smtp = new-object Net.Mail.SmtpClient($smtpServer)
 
 $msg.From = "someone@mydomain.co.uk"
 $msg.To.Add("me@mydomain.co.uk")
 $msg.Subject = "Citrix License Server Stats"
 $msg.Body = "The below is the current status of the license server:`n`nLicense Type: $LicTypeText`n`nInstalled Licences: $InstalledLicNum`n`nLicences In Use: $InUseNum`n`nPercentage: $PercentageNum%"
 
 $smtp.Send($msg)
 }
 
 $licserver = "mylicserver.mydomain.co.uk"
 $tempfile = "c:\lictest.txt"
 
 $webClient = New-Object System.Net.WebClient
 $webClient.credentials = New-Object system.net.networkcredential("usernametoaccesssite", "Password")
 $webadd = "http://$licserver/lmc/current_usage/currentUsage.jsp"
 $webClient.DownloadString($webadd) > $tempfile
 
 $Myline = Select-String -Path "$tempfile" -pattern "Enterprise"
 $LicTypeLine = $Myline.LineNumber - 1
 $InstalledLicLine = $LicTypeLine + 3
 $InUseLine = $InstalledLicLine + 1
 $PercentageLine = $LicTypeLine +6
 
 
 $LicTypeRAW = @(gc $tempfile)[$LicTypeLine]
 $LicTypeText = [regex]::match($LicTypeRAW,'(?<=).+(?=)').value
 
 $InstalledLicRAW = @(gc $tempfile)[$InstalledLicLine]
 $InstalledLicNum = [regex]::match($InstalledLicRAW,'(?<=).+(?=)').value
 
 $InUseRAW = @(gc $tempfile)[$InUseLine]
 $InUseNum = [regex]::match($InUseRAW,'(?<=).+(?=)').value
 
 $PercentageRAW = @(gc $tempfile)[$PercentageLine]
 $PercentageNum = [regex]::match($PercentageRAW,'[0-9]+').value
 
 if ($PercentageNum -lt 90)
 {
 }
 else
 {
 Sendemail $LicTypeText $InstalledLicNum $InUseNum $PercentageNum
 }
 
 if ($sendmailsched -eq "send")
 {
 Sendemail $LicTypeText $InstalledLicNum $InUseNum $PercentageNum
 }
 
 Remove-Item $tempfile
`

