---
Author: cglessner
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 1674
Published Date: 
Archived Date: 2010-03-09t09
---

# manage asp.net providers - 

## Description

manage asp.net membership, role and profile provider with powershell. especially useful with sqlmembershipprovider. i use it to manage sharepoint users with form based authentication (fba).

## Comments



## Usage



## TODO



## function

``

## Code

`#
 #
 
 param($appConfigPath=$null)
 
 [System.AppDomain]::CurrentDomain.SetData("APP_CONFIG_FILE", $appConfigPath )
 [void][System.Reflection.Assembly]::LoadWithPartialName("System.Web") 
 
 function global:Get-MembershipProvider($providerName=$null, [switch]$all)
 {    
 	if($all)
 	{
 		return [System.Web.Security.Membership]::Providers
 	}
 	
     if($providerName -eq $null)
     {
         return [System.Web.Security.Membership]::Provider
     }
     else
     {
         return [System.Web.Security.Membership]::Providers[$providerName]
     } 
 }
 
 function global:Add-MembershipUser($login=$(throw "-login is required"), $password=$(throw "$password is required"), $mail=$(throw "-mail is required"),$question, $answer, $approved=$true)
 {
 	$provider = $input | select -First 1
 	
 	if($provider -isnot [System.Web.Security.MembershipProvider])
 	{
 		$provider = Get-MembershipProvider
 	}
 
 	$status = 0
 	$provider.CreateUser($login, $password, $mail, $question, $answer, $approved, $null, [ref]$status)
 	return [System.Web.Security.MembershipCreateStatus]$status			
 }
 
 function global:Get-MembershipUser($identifier, $maxResult=100)
 {
 	$provider = $input | select -First 1
 
 	if($provider -isnot [System.Web.Security.MembershipProvider])
 	{
 		$provider = Get-MembershipProvider
 	}
 			
 	if($identifier -ne $null)
 	{		
 		$name = $provider.GetUserNameByEmail($identifier)
 		
 		if($name -ne $null){$identifier = $name}		
 		
 		return $provider.GetUser($identifier,$false)
 	}
 
 	$totalUsers = 0
 	$users = $provider.GetAllUsers(0,$maxResult,[ref]$totalUsers) 
 	
 	$users
 	
 	if($totalUsers -gt $maxResult)
 	{
 		throw "-maxResult limit exceeded"
 	}			
 }
 
 function global:Reset-MembershipUserPassword($identifier=$(throw "-identifier is required"), $questionAnswer)
 {
 	$provider = $input | select -First 1
 
 	if($provider -isnot [System.Web.Security.MembershipProvider])
 	{
 		$provider = Get-MembershipProvider
 	}
 	
 	$name = $provider.GetUserNameByEmail($identifier)
 		
 	if($name -ne $null){$identifier = $name}	
 	
 	return $provider.ResetPassword($identifier, $questionAnswer)
 }
 
 function global:Get-RoleProvider($providerName=$null, [switch]$all)
 {     
 	if($all)
 	{
 		return [System.Web.Security.Roles]::Providers
 	}
 
     if($providerName -eq $null)
     {
         return [System.Web.Security.Roles]::Provider
     }
     else
     {
         return [System.Web.Security.Roles]::Providers[$providerName]
     } 
 }
 
 function global:Get-ProfileProvider($providerName=$null)
 {     
 	if($all)
 	{
 		return [System.Web.Security.ProfileManager]::Providers
 	}
 
     if($providerName -eq $null)
     {
         return [System.Web.Profile.ProfileManager]::Provider
     }
     else
     {
         return [System.Web.Profile.ProfileManager]::Providers[$providerName]
     } 
 }
`

