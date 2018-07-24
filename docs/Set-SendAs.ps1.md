---
Author: jon webster
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 602
Published Date: 2008-09-23t23
Archived Date: 2012-12-15t05
---

# set-sendas - 

## Description

set the send as permissions on an exchange 2007 mailbox.

## Comments



## Usage



## TODO



## function

`set-sendas`

## Code

`#
 #
 #
 #
 
 Function Set-SendAs ($Identity, $SendAs, $ou, [Switch]$Remove, $DomainController = "dc01", [switch]$Confirm = $true)
 {
 	PROCESS
 	{
 		if(!$ou) {throw 'Required paramater $ou is missing.'}
 
 		if($_)
 		{
 			if($_.Identity -and !$Identity) {$Identity = $_.Identity}
 			if($_.SendAs -and !$SendAs) {$SendAs = $_.SendAs}
 			if($_.ou -and !$ou) {$ou = $_.ou}
 			if($_.remove -and !$remove) {$remove = $_.remove}
 		}
 		
 		$IdentityMbx = $Identity
 		$SendAsMbx = $SendAs
 
 		if([string]$Identity.GetType() -ne "Microsoft.Exchange.Data.Directory.Management.Mailbox" -and !($IdentityMbx = Get-Mailbox $Identity -DomainController $DomainController -OrganizationalUnit $ou -ErrorAction SilentlyContinue))
 		{throw "The operation could not be performed becaues the object '$Identity' could not be found in the organization '$ou'"}
 
 		if([string]$SendAs.GetType() -ne "Microsoft.Exchange.Data.Directory.Management.Mailbox" -and !($SendAsMbx = Get-Mailbox $SendAs -DomainController $DomainController -OrganizationalUnit $ou -ErrorAction SilentlyContinue))
 		{throw "The operation could not be performed becaues the object '$SendAs' could not be found in the organization '$ou'"}
 
 		If(!$Remove) {$null = Add-ADPermission -Identity $IdentityMbx.DistinguishedName -User $SendAsMbx.DistinguishedName -ExtendedRights Send-As}
 		Else {$null = Remove-ADPermission -Identity $IdentityMbx.DistinguishedName -User $SendAsMbx.DistinguishedName -ExtendedRights Send-As -Confirm:$Confirm}
 	}
 }
`

