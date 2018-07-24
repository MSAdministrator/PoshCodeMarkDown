---
Author: st3v3o
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2088
Published Date: 2011-08-18t07
Archived Date: 2017-04-30t10
---

# pwd expiration email - 

## Description

check to see if users passwords will expire in x days and send them an email notification.  this script was written using the active directory cmdlets bundled with server 2008 and powershell 2.0

## Comments



## Usage



## TODO



## module

``

## Code

`#
 #
 if(@(get-module | where-object {$_.Name -eq "ActiveDirectory"} ).count -eq 0) {import-module ActiveDirectory}
 
 
 $MaxPassAge = (Get-ADDefaultDomainPasswordPolicy).MaxPasswordAge.days
 
 if($MaxPassAge -le 0)
 
 { 
 
   throw "Domain 'MaximumPasswordAge' password policy is not configured."
 
 } 
 
 
 $DaysToExpire = 7
 
 $LogPath = "C:\Scripts\Logs\PasswordExpire"
 
 $a=get-date -format "ddMMyyyy"
 echo "Daily Log for $a" | Out-File $LogPath\$a.txt -append
 echo "-----------------------" | Out-File $LogPath\$a.txt -append
 
 
 Get-ADUser -SearchBase (Get-ADRootDSE).defaultNamingContext -Filter {(Enabled -eq "True") -and (PasswordNeverExpires -eq "False") -and (mail -like "*")} -Properties * | Select-Object Name,SamAccountName,mail,@{Name="Expires";Expression={ $MaxPassAge - ((Get-Date) - ($_.PasswordLastSet)).days}} | Where-Object {$_.Expires -gt 0 -AND $_.Expires -le $DaysToExpire } | ForEach-Object {
 
 
 $SMTPserver = "exchange.yourdomain.com"
 
 $from = "noreply@yourdomain.com"
 
 $to = $_.mail
 
 $subject = "Password reminder: Your Windows password will expire in $($_.Expires) days"
 
 $emailbody = "Your Windows password for the account $($_.SamAccountName) will expire in $($_.Expires) days.  For those of you on a Windows machine, please press CTRL-ALT-DEL and click Change Password.  
 
 For all others, you can change it with a web browser by using this link: https://yourdomain.com/owa/?ae=Options&t=ChangePassword
 
 Please remember to also update your password everywhere that might use your credentials like your phone or instant messaging application. 
 
 If you need help changing your password please contact helpdesk@yourdomain.com"
 
 
 $mailer = new-object Net.Mail.SMTPclient($SMTPserver)
 
 $msg = new-object Net.Mail.MailMessage($from, $to, $subject, $emailbody)
 
 $mailer.send($msg) 
 
 echo $($_.mail)  | Out-File $LogPath\$a.txt -append
 
 }
`

