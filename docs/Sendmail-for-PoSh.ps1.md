---
Author: patrick
Publisher: 
Copyright: 
Email: 
Version: 1.0
Encoding: ascii
License: cc0
PoshCode ID: 936
Published Date: 2009-03-12t08
Archived Date: 2016-05-20t03
---

# sendmail for posh - 

## Description

sending mails via powershell with text edit, signature support and encryption.

## Comments



## Usage



## TODO



## function

``

## Code

`#
 #
 $global:maileditor = "C:\Programme\vim\vim72\vim.exe"
 $global:encryption = "C:\Programme\GNU\GnuPG\gpg.exe"
 $global:enckey = "s.patrick1982@gmail.com"
 $global:tempmail = "C:\temp\psmail.txt"
 $global:sigmail = "C:\temp\halten\sig.txt"
 $global:mailbody = ""
 
 function global:initHeader {
 	$global:mailTo = ($global:mailTo).Split(',') | % { $_.Trim() }
 
 	if (Test-Path $global:sigmail) {
 		(Get-Content $global:sigmail | Out-String) | Out-File $global:tempmail
 	}
 
 	& $global:maileditor $global:tempmail
 
 }
 
 function global:initMail {
 	$smtpserver = "yoursmtpserver"
 	$myuser = "user"
 	$mypass = "pass"
 	$myAddress = "Patrick<s.patrick1982@gmail.com>"
 	
 	$global:mail = New-Object System.Net.Mail.MailMessage
 	$global:srv = New-Object System.Net.Mail.SmtpClient
 	$global:srv.Host = $smtpserver
 	$global:srv.Credentials = New-Object System.Net.NetworkCredential($myuser, $mypass)
 	
 	$global:mail.from = $myAddress
 	foreach ($rcpt in $global:mailTo) {
 		if ($rcpt -ne "") {
 			$global:mail.To.Add($rcpt)
 		}
 	}
 	$global:mail.subject = $global:subject
 }
 
 function global:initMailol {
 	$global:outlook = New-Object -ComObject Outlook.Application
 	$global:srv = New-Object -ComObject Redemption.SafeMailItem
 	$global:omail = $outlook.CreateItem("olMailItem")
 	$global:srv.Item = $global:omail
 
 	foreach ($rcpt in $global:mailTo) {
 		$global:srv.Recipients.Add($rcpt) | Out-Null
 	}
 
 	$check = $global:srv.Recipients.ResolveAll()
 	
 	if ($check -eq $False) {
 		for ($i=0; $i -gt $global:sitem.Recipients.Count;$i++) {
 			$global:srv.Recipients.Remove($i)
 			exit(-1)
 		}
 	}
 
 	$global:srv.item.Subject = $global:subject
 
 	if ($global:debug -eq $true) {
 		Write-Host "Mail an - " $global:sitem.Recipients
 		Write-Host "Betreff - " $global:subject
 	}
 }
 
 function global:sendmail {
 	Param (
 		[string]$global:client,
 		[string]$global:mailTo,
 		[string]$global:subject,
 		[string]$global:mtype
 	)
 
 	if (!$global:client) { 
 		$global:client = Read-Host -Prompt "Which Client (srv,ol): "
 	}
 	
 	if (!$global:mailto) {
 		$global:mailTo = Read-Host -Prompt "E-Mail To: "
 	}
 	
 	if (!$global:subject) {
 		$global:subject = Read-Host -Prompt "Subject: "
 	}
 	
 	initheader($global:mailTo, $global:subject)
 	
 	switch ($global:mtype) {
 		s {
 			& $global:encryption -a -r $global:enckey --clearsign $global:tempmail
 			break;
 		}
 		
 		e {
 			& $global:encryption -a -r $global:enckey --encrypt $global:tempmail 
 			break;
 		}
 	}
 
 	if ($global:mtype -ne "") {
 		$global:mailbody = (Get-Content $global:tempmail".asc" | Out-String)
 	} else {
 		$global:mailbody = (Get-Content $global:tempmail | Out-String)
 	}		
 	
 	if (test-path $global:tempmail) { Remove-Item $global:tempmail -Confirm:$false }
 	if (test-path $global:tempmail".asc" ) { Remove-Item $global:tempmail".asc" -Confirm:$false }
 	
 	switch ($global:client) {
 		srv {
 			initMail
 			$global:mail.Body = $global:mailbody
  			$global:srv.Send($global:mail)
 			break;
 		}
 		
 		ol {
 			initMailol
 			$global:srv.Body = $global:mailbody
 			$global:srv.Send()
 			break;
 		}
 	}
 
 }
`

