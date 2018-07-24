---
Author: george mauer
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2578
Published Date: 2012-03-24t14
Archived Date: 2017-03-01t06
---

# get-mailattachment.ps1 - 

## Description

get an attachment from an exchange e-mail.

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 param([Microsoft.Exchange.WebServices.Data.FileAttachment]$attachment)
 "Downloading Attachment"
 $attachment.Load()
 "Done"
 $path = "C:\temp\"+$attachment.Name
 "Writing to $path"
 set-content -value $mm[1].Attachments[0].Content -enc byte -path $path
 "Done"
 ii $path
`

