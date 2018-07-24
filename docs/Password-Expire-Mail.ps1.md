---
Author: dollarunderscore
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5052
Published Date: 2014-04-05t12
Archived Date: 2014-05-31t08
---

# password expire mail - 

## Description

this script sends out “password is about to expire” notifications by e-mail. it can send out custom mailmessage depending on where your users are in active directory. this makes it possible to for example send out different instructions for password changes, or have them written in different languages.

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 #========================================================================
 #========================================================================
 
 
 $FirstPasswordWarningDays = 14
 
 $SecondPasswordWarningDays = 7
 
 $LastPasswordWarningDays = 3
 
 $SMTPServer = "MySMTP.Contoso.Com"
 
 $PasswordExpiresLength = (Get-ADDefaultDomainPasswordPolicy).MaxPasswordAge
 
 $CurrentPWChangeDateLimit = (Get-Date).AddDays(-$PasswordExpiresLength.Days)
 
 $FirstPasswordDateLimit = $CurrentPWChangeDateLimit.AddDays($FirstPasswordWarningDays)
 $SecondPasswordDateLimit = $CurrentPWChangeDateLimit.AddDays($SecondPasswordWarningDays)
 $LastPasswordDateLimit = $CurrentPWChangeDateLimit.AddDays($LastPasswordWarningDays)
 
 $MailUsers = Get-ADUser -Filter "(Mail -like '*@*') -AND `
                                 (PasswordLastSet -le '$FirstPasswordDateLimit' -AND PasswordLastSet -gt '$($FirstPasswordDateLimit.AddDays(-1))' -OR `
                                 PasswordLastSet -le '$SecondPasswordDateLimit' -AND PasswordLastSet -gt '$($SecondPasswordDateLimit.AddDays(-1))' -OR `
                                 PasswordLastSet -le '$LastPasswordDateLimit' -AND PasswordLastSet -gt '$($LastPasswordDateLimit.AddDays(-1))') -AND `
                                 (PasswordNeverExpires -eq '$false' -AND Enabled -eq '$true')" -Properties PasswordLastSet, DisplayName, PasswordNeverExpires, mail
 
 foreach ($MailUser in $MailUsers) {
 
         $PasswordExpiresInDays = [System.Math]::Round((New-TimeSpan -Start $CurrentPWChangeDateLimit -End ($MailUser.PasswordLastSet)).TotalDays)
 
         Write-Output "$($MailUser.DisplayName) needs to change password in $PasswordExpiresInDays days."
 
 
         if ($MailUser.DistinguishedName -like "*MyOU1*") {
             $Subject = "Your password is expiring in $PasswordExpiresInDays days"
             $Body = "Hi $($MailUser.DisplayName),<BR><BR>Your password is expiring in $PasswordExpiresInDays days. Please change it now!<BR><BR>Don't forget to change it in your mobile devices if you are using mailsync.<BR><BR>Helpdesk 1"
             $EmailFrom = "Helpdesk 1 <no-reply@contoso.com>"
         }
         elseif ($MailUser.DistinguishedName -like "*MyOU2*") {
             $Subject = "Your password is expiring in $PasswordExpiresInDays days"
             $Body = "Hi $($MailUser.DisplayName),<BR><BR>Your password is expiring in $PasswordExpiresInDays days. Please change it now!<BR><BR>Don't forget to change it in your mobile devices if you are using mailsync.<BR><BR>Helpdesk 2"
             $EmailFrom = "Helpdesk 2 <no-reply@contoso.com>"
         }
         else {
             $Subject = "Your password is expiring in $PasswordExpiresInDays days"
             $Body = "Hi $($MailUser.DisplayName),<BR><BR>Your password is expiring in $PasswordExpiresInDays days. Please change it now!<BR><BR>Don't forget to change it in your mobile devices if you are using mailsync.<BR><BR>Helpdesk 3"
             $EmailFrom = "Helpdesk 3 <no-reply@contoso.com>"
         }
 
 
         Send-MailMessage -Body $Body -From $EmailFrom -SmtpServer $SMTPServer -Subject $Subject -Encoding UTF8 -BodyAsHtml -To $MailUser.mail
 
 }
`

