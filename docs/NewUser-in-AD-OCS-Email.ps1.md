---
Author: sean kearney
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 1491
Published Date: 2012-11-26t10
Archived Date: 2012-10-25t18
---

# newuser in ad/ocs/email - 

## Description

a powershell script meant for a novice to understand how to work with variables.  i use it daily.  creates a user via exchange 2007 and automatically populates the email address by defined policy.  using supplied scriptlets (referenced in ps1 file) it also populates the users’ info in ocs 2007 r2 standard as well as populates all fields in a/d with pertinent info.  it also sets up the display name in a lastname, firstname format and creates the users home folder with permissions allocated to the user only.   the only information it requests is firstname, lastname and password.  it is easy to modify to make it a bulk user setup.  thank you jeffrey snover and microsoft for powershell! the manna to administrators everywhere

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 
 
 $FirstName = read-host -Prompt "Enter User First Name: " 
 $LastName = read-host -Prompt "Enter User Last Name: " 
 
 
 $TempPassword = read-host -AsSecureString -Prompt "Please Enter Temporary Password" 
 
 
 $Sam=$FirstName+"."+$LastName 
 
 $max=$Sam.Length 
 
 
 if ($max -gt 20) {$max=20} 
 
 $Sam=$Sam.Substring(0,$max) 
 
 
 $Name=$Lastname+", "+$FirstName 
 $DisplayName=$Lastname+", "+$FirstName 
 
 
 $Alias=$FirstName+"."+$LastName 
 
 
 $UPN=$FirstName+"."+$LastName+"@Contoso.local" 
 
 
 $HomeDir='\\CONTOSOFILE\USERHOME$\'+$Alias 
 
 
 $HomeDrive='Z:' 
 
 
 $Phone='212-555-0000 x111' 
 
 
 $PostalZip='90210' 
 
 
 $City='Toronto' 
 
 
 $StateProv='Ontario' 
 
 
 $Address='123 Sesame Street' 
 
 
 $Web='www.contosorocks.com' 
 
 
 $Company='Contoso Rocks Ltd' 
 
 
 $Office='In the Basement with my stapler' 
 
 
 $Description='New User' 
 
 
 $JobTitle='New User Hired' 
 
 
 $Department='New Department Hire' 
 
 
 $Fax='212-555-1234' 
 
 
 $ourdomain='@contoso.local' 
 
 
 New-Mailbox -Name $Name -Alias $Alias -OrganizationalUnit 'Contoso.local/Users' -UserPrincipalName $UPN -SamAccountName $SAM -FirstName $FirstName -Initials '' -LastName $LastName -Password $TempPassword -ResetPasswordOnNextLogon $true -Database 'CONTOSOEXCHANGE\First Storage Group\Mailbox Database' 
 
 
 set-qaduser -identity $alias -homedirectory $HomeDir -homedrive $Homedrive -city $City -company $Company -department $Department -fax $Fax -office $Office -phonenumber $Phone -postalcode $PostalZip -stateorprovince $StateProv -streetaddress $Address -webpage $web -displayname $displayname -title $JobTitle 
 
 
 $SIPHOMESERVER='CN=LC Services,CN=Microsoft,CN=CONTOSO-OCSSERVER,CN=Pools,CN=RTC Service,CN=Microsoft,CN=System,DC=CONTOSO,DC=local' 
 
 $oa = @{'msRTCSIP-OptionFlags'=384; 'msRTCSIP-PrimaryHomeServer'=$SIPHOMESERVER; 'msRTCSIP-PrimaryUserAddress'=("sip:"+$alias+$ourdomain); 'msRTCSIP-UserEnabled'=$true } 
 
 Set-QADUser $Alias -oa $oa 
 
 
 
 $HomeFolderMasterDir='\\CONTOSOFILE\USERHOME$\' 
 
 new-item -path $HomeFolderMasterDir -name $Alias -type directory 
 
 $Foldername=$HomeFolderMasterDir+$Alias 
 $DomainUser='CONTOSO\'+$Alias 
 
 $ACL=Get-acl $Foldername 
 $Ar = New-Object  system.security.accesscontrol.filesystemaccessrule($DomainUser,"FullControl","Allow") 
 $Acl.SetAccessRule($Ar) 
 Set-Acl $Foldername $Acl
`

