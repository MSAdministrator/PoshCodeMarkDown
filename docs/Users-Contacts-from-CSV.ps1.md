---
Author: ken knicker
Publisher: 
Copyright: 
Email: 
Version: 1.00
Encoding: ascii
License: cc0
PoshCode ID: 1326
Published Date: 
Archived Date: 2010-07-17t01
---

# users/contacts from csv - 

## Description

creates or updates mail-enabled users or contact objects. developed to work within an exchange 2003 / windows server 2003 environment. depends on quest active directory extensions.

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 #=============================================================================#
 #=============================================================================#
 #=============================================================================#
 #
 #
 #
 #
 #
 #
 #
 #
 #
 #
 #
 #=============================================================================#
 #=============================================================================#
 #=============================================================================#
 
 
 #=============================================================================#
 #=============================================================================#
 #=============================================================================#
 #=============================================================================#
 
 
 #=============================================================================#
 Function Set_People_Attributes {
 #=============================================================================#
 	$Input | Set-QADObject -ObjectAttributes @{
 		c 					= ($_.c);
 		co 					= ($_.co);
 		countryCode 				= ($_.countryCode);
 		comment 				= ($_.comment);
 		company 				= ($_.company);
 		description 				= ($_.description);
 		division 				= ($_.division);
 		employeeID 				= ($_.employeeID);
 		employeeType 				= ($_.employeeType);
 		extensionAttribute1			= ($_.extensionAttribute1);
 		extensionAttribute2			= ($_.extensionAttribute2);
 		extensionAttribute3			= ($_.extensionAttribute3);
 		extensionAttribute4			= ($_.extensionAttribute4);
 		extensionAttribute5			= ($_.extensionAttribute5);
 		extensionAttribute6			= ($_.extensionAttribute6);
 		extensionAttribute7			= ($_.extensionAttribute7);
 		extensionAttribute8			= ($_.extensionAttribute8);
 		extensionAttribute9			= ($_.extensionAttribute9);
 		extensionAttribute10			= ($_.extensionAttribute10);
 		extensionAttribute11			= ($_.extensionAttribute11);
 		extensionAttribute12			= ($_.extensionAttribute12);
 		extensionAttribute13			= ($_.extensionAttribute13);
 		extensionAttribute14			= ($_.extensionAttribute14);
 		extensionAttribute15			= ($_.extensionAttribute15);
 		facsimileTelephoneNumber		= ($_.facsimileTelephoneNumber);
 		givenName				= ($_.givenName);
 		info					= ($_.info);
 		initials				= ($_.initials);
 		l					= ($_.l);
 		mobile					= ($_.mobile);
 		physicalDeliveryOfficeName		= ($_.physicalDeliveryOfficeName);
 		postalCode				= ($_.postalCode);
 		sn					= ($_.sn);
 		st					= ($_.st);
 		streetAddress				= ($_.streetAddress);
 		telephoneNumber				= ($_.telephoneNumber);
 		title					= ($_.title)
 	} -Verbose
 }
 
 
 #=============================================================================#
 Function Set_User_Attributes {
 #=============================================================================#
 	$Input | Set-QADUser -ObjectAttributes @{
 		homeDirectory 				= ($_.homeDirectory);
 		homeDrive				= ($_.homeDrive);
 		mobile					= ($_.mobile);
 		profilePath				= ($_.profilePath);
 		sAMAccountName				= ($_.sAMAccountName);
 		scriptPath 				= ($_.scriptPath);
 		userPrincipalName 			= ($_.sAMAccountName + $Domain);
 		wWWHomePage				= ($_.wWWHomePage)
 	} -Verbose
 }
 
 
 #=============================================================================#
 Function Set_Mail_Attributes {
 #=============================================================================#
 	$Input | Set-QADObject -ObjectAttributes @{
 		legacyExchangeDN 			= ($Root_LegacyExchangeDN + $_.mailNickname);
 		mail 					= ($_.mail);
 		mailNickname				= ($_.mailNickname);
 		targetAddress 				= ($_.targetAddress)
 	} -Verbose
 }
 
 
 #=============================================================================#
 #=============================================================================#
 #=============================================================================#
 #=============================================================================#
 
 
 #=============================================================================#
 #=============================================================================#
 If (-not (Get-PSSnapin Quest.ActiveRoles.ADManagement -ErrorAction SilentlyContinue)) {
 Add-PSSnapin Quest.ActiveRoles.ADManagement -Verbose
 }
 
 
 #=============================================================================#
 #=============================================================================#
 $CSVPath 		= "\\SERVER\SharePath\ListOfPeople.csv"
 $MailEnable 	= $false
 
 $ObjectType						= $null
 $Domain							= $null
 $OU 							= $null
 $Root_LegacyExchangeDN					= $null
 $Server							= $null
 $UserName 						= $null
 
 #=============================================================================#
 #=============================================================================#
 $Affiliate1_Contact_OU					= "OU=Contacts,DC=affiliate1,DC=com"
 $Affiliate1_User_OU					= "OU=Users,DC=affiliate1,DC=com"
 $Affiliate1_Root_LegacyExchangeDN			= "/O=Affiliate 1/OU=AFFILIATE1/cn=Recipients/cn="
 $Affiliate1_Domain					= "@affiliate1.com"
 $Affiliate1_Server					= "domaincontroller.affiliate1.com"
 
 #=============================================================================#
 #=============================================================================#
 $Affiliate2_Contact_OU					= "OU=Contacts,DC=affiliate2,DC=com"
 $Affiliate2_User_OU					= "OU=Users,DC=affiliate2,DC=com"
 $Affiliate2_Root_LegacyExchangeDN			= "/O=Affiliate 2/OU=AFFILIATE2/cn=Recipients/cn="
 $Affiliate2_Domain					= "@affiliate2.com"
 $Affiliate2_Server					= "domaincontroller.affiliate2.com"
 
 #=============================================================================#
 #=============================================================================#
 $Location = Read-Host "Affiliate location: ''A1'' for Affiliate 1, ''A2'' for Affiliate 2:"
 $ObjectType = Read-Host "Object type? ''Contact'' or ''User'':"
 
 If ($Location -eq "AF1") {
 	If ($ObjectType -eq "User") { $OU = $Affiliate1_User_OU } 
 	Else { $OU = $CHI_Contact_OU }
 	$Root_LegacyExchangeDN				= $Affiliate1_Root_LegacyExchangeDN
 	$Domain						= $Affiliate1_Domain
 	$Server						= $Affiliate1_Server
 	$UserName 					= $Affiliate1_UserName
 }
 
 If ($Location -eq "AF2") {
 	If ($ObjectType -eq "User") { $OU = $Affiliate2_User_OU } 
 	Else { $OU = $CHI_Contact_OU }
 	$Root_LegacyExchangeDN				= $Affiliate2_Root_LegacyExchangeDN
 	$Domain						= $Affiliate2_Domain
 	$Server						= $Affiliate2_Server
 	$UserName 					= $Affiliate2_UserName
 }
 
 $UserName = Read-Host "Enter username for the domain you are connecting to (''DOMAIN\username''):"
 $Password = Read-Host "Enter password for"$UserName -AsSecureString
 Connect-QADService -Service $Server -ConnectionAccount $UserName -ConnectionPassword $Password -Verbose
 
 
 #=============================================================================#
 #=============================================================================#
 #=============================================================================#
 #=============================================================================#
 
 
 #=============================================================================#
 #=============================================================================#
 $PeopleList | ForEach-Object {
 
 	$Person = Get-QADObject -DisplayName $_.displayName -LdapFilter "(&(objectClass=$ObjectType))" -Verbose
 
 	If ($Person -eq $Null){
 		Write-Host "Creating Object for" $_.displayName -ForegroundColor Green -Verbose
 		If ($ObjectType -eq "Contact") {
 			$Person = New-QADObject -ParentContainer $OU -Name $_.cn -Type $ObjectType -DisplayName $_.displayName -Verbose
 		}
 		ElseIf ($ObjectType -eq "User") {
 			$Person = New-QADUser -ParentContainer $OU -Name $_.cn -DisplayName $_.displayName -SamAccountName $_.sAMAccountName -UserPrincipalName ($_.sAMAccountName+"$Domain") -Verbose
 			$Person | Set_User_Attributes
 		}
 		$Person | Set_People_Attributes
 	}
 	
 	Else {
 		Write-Host "Updating Properties for" $_.displayName -ForegroundColor Red -Verbose
 		$Person | Set_People_Attributes
 	}
 	If ($MailEnable -eq $true) {
 		$Person | Set_Mail_Attributes
 	}
 }
 
 #=============================================================================#
 #=============================================================================#
`

