---
Author: sean kearney
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 1521
Published Date: 2010-12-11t13
Archived Date: 2015-07-18t04
---

# set single email - 

## Description

script which creates a new mail enabled user with the email address policy disabled and a single custom email address attached to the account.  designed for new powershell users.

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 #
 #
 #
 #
 #
 #
 #
 $FirstName=READ-HOST 'Enter First Name '
 $LastName=READ-HOST 'Enter Last Name'
 #
 #
 $EmailAddress=READ-HOST 'E-mail Address reqd '
 #
 #
 $Password=READ-HOST 'DefaultPassword' -assecurestring
 #
 #
 $Name=$FirstName+' '+$LastName
 $Alias=$Firstname+'.'+$LastName
 $OU='Contoso.local/Offices/Seattle'
 $UPN=$Alias+'@contoso.local'
 #
 #
 $FirstName
 $LastName
 $EmailAddress
 
 $Password
 $Name
 $Alias
 $OU
 $UPN
 
 #
 #
 New-Mailbox -Name $Name -Alias $Alias -OrganizationalUnit $OU -UserPrincipalName $UPN -SamAccountName $Firstname -FirstName $FirstName -Initials '' -LastName $Lastname -Password $Password -ResetPasswordOnNextLogon $false -Database 'OUREXCHANGESERVER\First Storage Group\Mailbox Database'
 #
 #
 #
 get-mailbox $Alias | Set-Mailbox -EmailAddressPolicyEnabled $False
 #
 #
 $mailboxuser=get-mailbox $Alias
 $mailboxuser.EmailAddresses.Clear()
 #
 #
 $mailboxuser.EmailAddresses.Add($EmailAddress)
 #
 #
 $mailboxuser | set-mailbox
`

