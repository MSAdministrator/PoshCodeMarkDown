---
Author: paul brice
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5431
Published Date: 2016-09-16t13
Archived Date: 2016-05-31t11
---

# exporting sqldata to csv - 

## Description

“taking data from sql [into] csv [ehlo] smtp-email”

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 $Database = "Database"
 $Server = "SQLServer"
 $SMTPServer = "smtp.domain.com"
 $AttachmentPath = "C:\SQLData.csv"
 $SqlQuery = "SELECT * FROM dbo.Test_Table"
 $SqlConnection = New-Object System.Data.SqlClient.SqlConnection
 $SqlConnection.ConnectionString = "Data Source=$Server;Initial Catalog=$Database;Integrated Security = True"
 $SqlCmd = New-Object System.Data.SqlClient.SqlCommand
 $SqlCmd.CommandText = $SqlQuery
 $SqlCmd.Connection = $SqlConnection
 $SqlAdapter = New-Object System.Data.SqlClient.SqlDataAdapter
 $SqlAdapter.SelectCommand = $SqlCmd
 $DataSet = New-Object System.Data.DataSet
 $nRecs = $SqlAdapter.Fill($DataSet)
 $nRecs | Out-Null
 $objTable = $DataSet.Tables[0]
 $objTable | Export-CSV $AttachmentPath
 $Mailer = new-object Net.Mail.SMTPclient($SMTPServer)
 $From = "email1@domain.com"
 $To = "email2@domain.com"
 $Subject = "Test Subject"
 $Body = "Body Test"
 $Msg = new-object Net.Mail.MailMessage($From,$To,$Subject,$Body)
 $Msg.IsBodyHTML = $False
 $Attachment = new-object Net.Mail.Attachment($AttachmentPath)
 $Msg.attachments.add($Attachment)
 $Mailer.send($Msg)
`

