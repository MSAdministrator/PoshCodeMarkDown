---
Author: jeff patton
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 3746
Published Date: 2012-11-04t21
Archived Date: 2012-11-07t23
---

# get-adgroupmembers - 

## Description

this function returns an object that contains all the properties of a user object. this function works for small groups as well as groups in excess of 1000.

## Comments



## Usage



## TODO



## function

`get-adgroupmembers`

## Code

`#
 #
 Function Get-ADGroupMembers
 {
     <#
         .SYNOPSIS
             Return a collection of users in an ActiveDirectory group.
         .DESCRIPTION
             This function returns an object that contains all the properties of a user object. This function
             works for small groups as well as groups in excess of 1000.
         .PARAMETER UserGroup
             The name of the group to get membership from.
         .PARAMETER UserDomain
             The LDAP URL of the domain that the group resides in.
         .EXAMPLE
             Get-ADGroupMembers -UserGroup Managers |Format-Table -Property name, distinguishedName, cn
 
             name                             distinguishedName                cn                              
             ----                             -----------------                --                              
             {Steve Roberts}                  {CN=Steve Roberts,CN=Users,DC... {Steve Roberts}                 
             {S-1-5-21-57989841-1078081533... {CN=S-1-5-21-57989841-1078081... {S-1-5-21-57989841-1078081533...
             {S-1-5-21-57989841-1078081533... {CN=S-1-5-21-57989841-1078081... {S-1-5-21-57989841-1078081533...
             {Matt Temple}                    {CN=Matt Temple,CN=Users,DC=c... {Matt Temple}                   
             ...
             Description
             -----------
             This example shows passing in a group name, but leaving the default domain name in place.
         .NOTES
             The context under which this script is run must have rights to pull infromation from ActiveDirectory.
         .LINK
     #>
     Param
         (
     $UserGroup = "Domain Users",
     [ADSI]$UserDomain = ("LDAP://DC=company,DC=com")
         )
 
     Begin
         {
             $DirectoryEntry = New-Object System.DirectoryServices.DirectoryEntry($UserDomain.Path)
             $DirectorySearcher = New-Object System.DirectoryServices.DirectorySearcher
 
             $LDAPFilter = "(&(objectCategory=Group)(name=$($UserGroup)))"
 
             $DirectorySearcher.SearchRoot = $DirectoryEntry
             $DirectorySearcher.PageSize = 1000
             $DirectorySearcher.Filter = $LDAPFilter
             $DirectorySearcher.SearchScope = "Subtree"
 
             $SearchResult = $DirectorySearcher.FindAll()
             
             $UserAccounts = @()
         }
 
     Process
         {
             foreach ($Item in $SearchResult)
             {
                 $Group = $Item.GetDirectoryEntry()
                 $Members = $Group.member
                 
                 If ($Members -ne $Null)
                 {
                     foreach ($User in $Members)
                     {
                         $UserObject = New-Object System.DirectoryServices.DirectoryEntry("LDAP://$($User)")
                         If ($UserObject.objectCategory.Value.Contains("Group"))
                         {
                         }
                         Else
                         {
                             $ThisUser = New-Object -TypeName PSObject -Property @{
                                 cn = $UserObject.cn
                                 distinguishedName = $UserObject.distinguishedName
                                 name = $UserObject.name
                                 nTSecurityDescriptor = $UserObject.nTSecurityDescriptor
                                 objectCategory = $UserObject.objectCategory
                                 objectClass = $UserObject.objectClass
                                 objectGUID = $UserObject.objectGUID
                                 objectSID = $UserObject.objectSID
                                 showInAdvancedViewOnly = $UserObject.showInAdvancedViewOnly
                             }
                         }
                     $UserAccounts += $ThisUser
                     }
                 }
             }
         }
 
     End
         {
             Return $UserAccounts
         }
 }
`

