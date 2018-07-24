---
Author: george mauer
Publisher: 
Copyright: 
Email: 
Version: 1.1
Encoding: ascii
License: cc0
PoshCode ID: 6573
Published Date: 2016-10-13t05
Archived Date: 2016-12-22t03
---

# get-exchange-mail.ps1 - 

## Description

connect to an exchange mailbox and get your latest emails.

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 [Reflection.Assembly]::LoadFile("C:\Program Files\Microsoft\Exchange\Web Services\1.1\Microsoft.Exchange.WebServices.dll") | Out-Null
 $s = New-Object Microsoft.Exchange.WebServices.Data.ExchangeService([Microsoft.Exchange.WebServices.Data.ExchangeVersion]::Exchange2007_SP1)
 $s.Credentials = New-Object Net.NetworkCredential('email@domain', 'password')
 $s.Url = new-object Uri("https://red001.mail.microsoftonline.com/ews/exchange.asmx")
 
 $inbox = [Microsoft.Exchange.WebServices.Data.Folder]::Bind($s,[Microsoft.Exchange.WebServices.Data.WellKnownFolderName]::Inbox)
 $mails = $inbox.FindItems(5) 
 $mails | % {$_.Load()}
 $mails
`

