---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 570
Published Date: 
Archived Date: 2016-03-10t00
---

#  - 

## Description

code to auto update the address policy, gal, oab and storage groups and mailbox databases of an exchange 2007 server

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
 #################################################################################################################################
 
 param([string]$fullname, [string]$mail2, [string]$typeofcustomer)
 
 
 
 
 ######################
 
 Add-PSSnapin Microsoft.Exchange.Management.PowerShell.Admin
 
 new-AcceptedDomain -Name $mail2 -DomainName $mail2 -DomainType 'Authoritative'
 
 new-EmailAddressPolicy -Name $fullname -IncludedRecipients 'AllRecipients' -ConditionalCompany $fullname -Priority 'Lowest' -EnabledEmailAddressTemplates SMTP:%m@$mail2,smtp:%g@$mail2,smtp:%1g.%s@$mail2,smtp:%m@$yourinternaldomain
 
 set-EmailAddressPolicy -Identity $fullname -IncludedRecipients 'AllRecipients' -ConditionalCompany $fullname -Priority 'Lowest' -EnabledEmailAddressTemplates SMTP:%m@$mail2,smtp:%g@$mail2,smtp:%1g.%s@$mail2,smtp:%m@$yourinternaldomain
 
 new-GlobalAddressList -Name $fullname -IncludedRecipients 'AllRecipients' -ConditionalCompany $fullname
 
 new-AddressList -Name $fullname -IncludedRecipients 'AllRecipients' -ConditionalCompany $fullname -Container '\Generic Address Book'
 
 $fullname2 = '\Generic Address book\' + $fullname
 
 $OABPoint = $OABdistributionServer + '\OAB (Default Web Site)'
 new-OfflineAddressBook -Name $fullname -Server $pinkyring -AddressLists $fullname2 -PublicFolderDistributionEnabled $false -VirtualDirectories $OABPoint
 Set-OfflineAddressBook -Identity $fullname -versions Version3, Version4 -PublicFolderDistributionEnabled $true -AddressLists $fullname2 
 
 Update-EmailAddressPolicy -Identity $fullname
 
 Set-MailboxDatabase -Identity $fullname -OfflineAddressBook $fullname -PublicFolderDatabase $PublicFolderDatabase
 
 $maildb = Get-MailboxDataBase | Select-String $fullname
 echo $maildb
 if ($maildb -match "sg") 
 	{
 	echo "not making the DB"
 	}
 	else
 	{
 	if ($typeofcustomer -match "3")
 		{
 		$i = 1
 
 			for( $i=0;  $i -lt 50;  $i++ )	
 			{
 			$maildb = Get-MailboxDataBase | Select-String $fullname
 			echo $maildb
 				if ($maildb -match "sg") 
 				else
 				{
 				new-storagegroup sg$i
 				new-mailboxdatabase -StorageGroup $MailboxServer\sg$i -Name $fullname -EdbFilePath $LogFilePath + '\' + $fullname.edb
 				mount-database -Identity "CN=$fullname,CN=sg$i,CN=InformationStore,CN=$MailboxServer,$AdminName"
 				$i++
 				}
 			}
 			Set-MailboxDatabase -Identity $fullname -PublicFolderDatabase $PublicFolderDatabase
 	}
 	else
 	{
 	echo "customer is a basic or standard customer so we are going to not create a DB"
 	Set-MailboxDatabase -Identity $fullname -OfflineAddressBook $fullname -PublicFolderDatabase $PublicFolderDatabase
 	}
 }
`

