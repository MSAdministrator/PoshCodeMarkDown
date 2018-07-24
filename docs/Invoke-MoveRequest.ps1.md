---
Author: jan egil ring
Publisher: 
Copyright: 
Email: 
Version: 1.0
Encoding: ascii
License: cc0
PoshCode ID: 1838
Published Date: 2010-05-13t06
Archived Date: 2016-08-22t17
---

# invoke-moverequest - invoke-moverequest.ps1

## Description

script to use when migrating mailboxes to microsoft exchange server 2010 cross-forest. prepares user objects already moved to the target forest using active directory migration tool or other tools, and then invokes the mailbox move request.

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 ###########################################################################
 #
 #
 #
 #
 #
 #
 ###########################################################################
 
 Add-PSSnapin Quest.ActiveRoles.ADManagement
 Add-PSSnapin Microsoft.Exchange.Management.PowerShell.E2010
 
 $TargetDatabase = "Mailbox Database A"
 $TargetDeliveryDomain = "domain.com"
 $TargetForest="DomainA.local"
 $SourceForest="domainB.local"
 $SourceForestGlobalCatalog = "dc-01.domainB.local"
 $SourceForestCredential = Get-Credential -Credential "Domain\SourceForestAdministrator"
 
 Connect-QADService -Service $SourceForest | Out-Null
 $SourceForestUsersToMigrate = Get-QADUser -SearchRoot "DomainA.local/Department A/Users"
 
 
 foreach ($user in $SourceForestUsersToMigrate) {
 
 Write-Host "Processing user $user" -ForegroundColor Yellow
 
 Connect-QADService -Service $SourceForest | Out-Null
 $TargetObject=$user.samaccountname
 $SourceObject=Get-QADUser $TargetObject -IncludeAllProperties
 $mail=$SourceObject.Mail
 $mailNickname=$SourceObject.mailNickname
 [byte[]]$msExchMailboxGUID=(Get-QADuser $SourceObject -IncludedProperties msExchMailboxGUID -DontConvertValuesToFriendlyRepresentation).msExchMailboxGUID
 $msExchRecipientDisplayType="-2147483642"
 $msExchRecipientTypeDetails="128"
 $msExchUserCulture=$SourceObject.msExchUserCulture
 $msExchVersion="44220983382016"
 $proxyAddresses=$SourceObject.proxyAddresses
 $targetAddress=$SourceObject.Mail
 $userAccountControl=514
 
 Connect-QADService -Service $TargetForest | Out-Null
 Set-QADUser -Identity $TargetObject -ObjectAttributes @{mail=$mail;mailNickname=$mailNickname;msExchMailboxGUID=$msExchMailboxGUID;msExchRecipientDisplayType=$msExchRecipientDisplayType;msExchRecipientTypeDetails=$msExchRecipientTypeDetails;msExchUserCulture=$msExchUserCulture;msExchVersion=$msExchVersion;proxyAddresses=$proxyAddresses;targetAddress=$targetAddress;userAccountControl=$userAccountControl} | Out-Null
 
 Get-MailUser $TargetObject | Update-Recipient
 
 New-MoveRequest -Identity $TargetObject -RemoteLegacy -TargetDatabase $TargetDatabase -RemoteGlobalCatalog $SourceForestGlobalCatalog -RemoteCredential $SourceForestCredential -TargetDeliveryDomain $TargetDeliveryDomain
 
 Enable-QADUser -Identity $TargetObject | Out-Null
 Set-QADUser -Identity $TargetObject -UserMustChangePassword $false | Out-Null
 }
`

