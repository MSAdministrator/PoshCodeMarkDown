---
Author: jan egil ring
Publisher: 
Copyright: 
Email: jan.egil.ring@powershell.no
Version: 1.0
Encoding: ascii
License: cc0
PoshCode ID: 1086
Published Date: 2009-05-10t07
Archived Date: 2016-06-01t12
---

# get-sqldatabasefreespace - get-sqldatabasefreespace.ps1

## Description

i created this script because of the need to monitor a msde 2000 database which have reached the 2 gb database limit until the application administrator deleted old data from the database.  sql server 2008 management studio express must be installed on the computer running the script as a scheduled task.

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 ###########################################################################"
 #
 #
 #
 #
 #
 #
 ###########################################################################"
 
 Add-Pssnapin SqlServerProviderSnapin100
 
 cd SQLSERVER:\SQL\SQL-server-name\Instance-name\Databases
 $DB =  Get-Item "Database01"
 $SpaceAvailable = $DB.SpaceAvailable
 $formattedresult = $SpaceAvailable
 $result = "{0:#.00}" -f ($SpaceAvailable/1kb)
 
 $smtpServer = "smtp-server-name" 
 $msg = new-object Net.Mail.MailMessage
 $smtp = new-object Net.Mail.SmtpClient($smtpServer)
 $msg.From = "sender@domain.local"
 $msg.To.Add("recipient@domain.local")
 $msg.Subject = "Free space in Database01"
 $msg.Body = "There are $result MB free space in Database01"
 $smtp.Send($msg)
`

