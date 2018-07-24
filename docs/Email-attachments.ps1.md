---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 636
Published Date: 
Archived Date: 2009-05-14t07
---

# email attachments - 

## Description

all descriptions on the web which show how to do this so far have left the email attachment open which means if the script is continuing after the email and you wish to use the file you have attached you will not be able to as it will show as locked, use this example to close the attached file correctly using .dispose()

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 $file = "MYFILE.TXT"
 
 $smtpServer = "MYSMTPSERVER.EMAIL.CO.UK"
 
 $msg = new-object Net.Mail.MailMessage
 $att = new-object Net.Mail.Attachment($file)
 $smtp = new-object Net.Mail.SmtpClient($smtpServer)
 
 $msg.From = "FROMME@EMAIL.CO.UK"
 $msg.To.Add("TOME@EMAIL.CO.UK")
 $msg.Subject = "MY SUBJECT"
 $msg.Body = "MY TEXT FOR THE EMAIL"
 $msg.Attachments.Add($att)
 
 $smtp.Send($msg)
 
 $att.Dispose()
`

