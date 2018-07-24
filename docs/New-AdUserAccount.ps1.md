---
Author: andy arismendi
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: utf-8
License: cc0
PoshCode ID: 6525
Published Date: 2016-09-23t11
Archived Date: 2016-09-28t18
---

# new-aduseraccount - 

## Description

here is a function to create an active directory user. this function doesn’t do nearly everything that the quest ad cmdlet can do, however it does provide user password configuration options such as setting the ‘user cannot change password’ and ‘password never expires’ flags. the new ad account is enabled by default when created.

## Comments



## Usage



## TODO



## function

`new-aduseraccount`

## Code

`#
 #
 function New-AdUserAccount {
     param (
         [string] $Username = $(throw "Parameter -Username [System.String] is required."),
         [string] $Password = $(throw "Parameter -Password [System.String] is required."),
         [string] $OrganizationalUnit = "Users",
         [string] $DisplayName,
         [string] $FirstName,
         [string] $LastName,
         [string] $Initials,
         [string] $Description,
         [switch] $CannotChangePassword,
         [switch] $PasswordNeverExpires,
         [switch] $Disabled
     )
     
     try {
         $currentDomain = [System.DirectoryServices.ActiveDirectory.Domain]::GetCurrentDomain()
         $dn = $currentDomain.GetDirectoryEntry().distinguishedName
         $ou = [ADSI] "LDAP://CN=$OrganizationalUnit,$dn"
         
         $userAccount = $ou.Create("user", "cn=$Username")
         $userAccount.SetInfo()
 
         $userAccount.SetInfo()
         
         $userAccount.sAMAccountName = $Username
         $userAccount.SetInfo()
         
         $userAccount.userPrincipalName = ("{0}@{1}" -f $Username, $currentDomain.Name)
         if ($DisplayName) {
             $userAccount.displayName = $DisplayName
         }
         
         if ($Description) {
             $userAccount.description = $Description
         }
         
         if ($FirstName) {
             $userAccount.givenname = $FirstName
         }
         
         if ($LastName) {
             $userAccount.sn = $LastName
         }
         
         if ($Initials) {
             $userAccount.initials = $Initials
         }
         $userAccount.SetInfo()
         
         $userAccount.SetPassword($Password)
         
         if ($PasswordNeverExpires) {
             $userAccount.userAccountControl = ($userAccount.userAccountControl.Item(0) -bxor 0x10000)
         }
         
         if ($CannotChangePassword) {
             $everyOne = [System.Security.Principal.SecurityIdentifier]'S-1-1-0' 
             $EveryoneDeny = new-object System.DirectoryServices.ActiveDirectoryAccessRule ($Everyone,'ExtendedRight','Deny', [System.Guid]'ab721a53-1e2f-11d0-9819-00aa0040529b')
              
             $self = [System.Security.Principal.SecurityIdentifier]'S-1-5-10' 
             $SelfDeny = new-object System.DirectoryServices.ActiveDirectoryAccessRule ($self,'ExtendedRight','Deny', [System.Guid]'ab721a53-1e2f-11d0-9819-00aa0040529b') 
 
             $userAccount.get_ObjectSecurity().AddAccessRule($selfDeny) 
             $userAccount.get_ObjectSecurity().AddAccessRule($EveryoneDeny) 
 
             $userAccount.CommitChanges() 
         }
         $userAccount.SetInfo()
         
         if ($Disabled) {
             $userAccount.userAccountControl = ($userAccount.userAccountControl.Item(0) -bxor 0x0002)
         }
         $userAccount.SetInfo()
                 
     } catch {
         Write-Error $_
                 
         #$ou.Delete("user", "cn=$Username")
         
         return $false
         
     }
     return $true
 }
 
 New-AdUserAccount -Username bob -Password 'B0bsP4$$W0rD!@#$%' -Description "Bob's account" -FirstName Bob -LastName Doe -DisplayName "Bob Doe" -Initials R -PasswordNeverExpires -CannotChangePassword -Disabled
`

