---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 1067
Published Date: 
Archived Date: 2009-05-03t11
---

# get-adnonexppass - 

## Description

this script will retrieve all user accounts whose passwords are set to not expire for a given ldap path. defaults to root of the domain.

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 
 param ($LDAPPath = "", [switch]$Help)
 
 if ($Help)
 {
 	""
 	Write-Host "Usage: .\Get-ADNonExpPass.ps1 <LDAPPath>" -foregroundcolor Yellow
 	Write-Host "Ex: .\Get-ADNonExpPass.ps1 'LDAP://ou=users,dc=domain,dc=com'" -foregroundcolor Yellow
 	""
 	break
 }
 
 $DontExpire = 0x10000
 
 $Root = [ADSI]$LDAPPath
 
 $Category = "user"
 
 $Selector = New-Object DirectoryServices.DirectorySearcher
 $Selector.SearchRoot = $Root 
 $Selector.Filter = ("(objectCategory=$Category)")
 #$Selector.pagesize = 2000
 
 $Users = $Selector.findall()
 
 foreach ($User in $Users) {
 
 	$DN = $User.properties.distinguishedname
 	$UserProp = [ADSI]"LDAP://$dn"
 	
 	if (($UserProp.UserAccountControl[0] -band $DontExpire) -eq 65536)
 		{
 		$UserProp | Select Name, UserAccountControl
 		}
 
 }
`

