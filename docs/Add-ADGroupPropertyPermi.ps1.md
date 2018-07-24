---
Author: dollarunderscore
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 4917
Published Date: 2014-02-20t17
Archived Date: 2014-05-31t09
---

# add-adgrouppropertypermi - 

## Description

this function sets the access rights on properties of ad-groups. used to delegate access on one group to another. in my case used to delegate the accessgroup of a shared mailbox in office 365 to another group which contains the owners. this enables them to control the members of the access group through a powershell form.

## Comments



## Usage



## TODO



## function

`add-adgrouppropertypermission`

## Code

`#
 #
 function Add-ADGroupPropertyPermission
 {
     <#
     .SYNOPSIS
     This function is used for setting access rights on properties on Active Directory Groups.
     Use this code with caution! It has not been tested on a lot of objects/properties/access rights!
 
     .DESCRIPTION
     This function changes the ACLs on AD-Groups to enable granular delegation of them to other groups.
 
     Use this code with caution! It has not been tested on a lot of objects/properties/access rights!
 
     .EXAMPLE
     Add-ADGroupPropertyPermission -ADObject TheGroupSomeoneWantsAccessTo -MasterObject TheGroupWhoWillGainAccess -AccessRight WriteProperty -AccessRule Allow -Property Member -ActiveDirectoryServer MyDomain
 
     .PARAMETER ADObject
     Specify the identity of the group you want to delegate to the other group.
 
     .PARAMETER MasterObject
     Specify the identity of the group who should gain access to the specified property.
 
     .PARAMETER AccessRight
     Specify what access should be added, for example WriteProperty.
 
     .PARAMETER AccessRule
     Set this to Allow or Deny.
 
     .PARAMETER Property
     Specify which property this should be applied for.
 
     .PARAMETER ActiveDirectoryServer
     Specify domain or domain controller where the search for the groups will take place.
 
     #>
 
     [cmdletbinding()]
     param (
            [Parameter(Mandatory=$True)]
            $ADObject,
            [Parameter(Mandatory=$True)]
            $MasterObject,
            [Parameter(Mandatory=$True)]
            $AccessRight,
            [Parameter(Mandatory=$True)]
            [ValidateSet("Allow","Deny")]
            $AccessRule,
            [Parameter(Mandatory=$True)]
            $Property,
            $ActiveDirectoryServer = $(Get-ADDomain | select -ExpandProperty DNSRoot))
 
     try {
         $TheAccessGroup = Get-ADGroup -Identity $ADObject -Server $ActiveDirectoryServer -ErrorAction Stop
     }
     catch {
         Write-Error "Failed to get the object with identity $ADObject. The error was: $($Error[0])."
         return
     }
 
     try {
         $TheOwnerGroup = Get-ADGroup -Identity $MasterObject -Server $ActiveDirectoryServer -ErrorAction Stop
     }
     catch {
         Write-Error "Failed to get the object with identity $MasterObject. The error was: $($Error[0])."
         return
     }
 
     try {
         $AccessGroupSid = New-Object System.Security.Principal.SecurityIdentifier ($TheAccessGroup).SID -ErrorAction Stop
     }
     catch {
         Write-Error "Failed to resolve the sid of $ADObject. The error was: $($Error[0])."
         return
     }
 
     try {
         $OwnerGroupSid = New-Object System.Security.Principal.SecurityIdentifier ($TheOwnerGroup).SID -ErrorAction Stop
     }
     catch {
         Write-Error "Failed to resolve the sid of $MasterObject. The error was: $($Error[0])."
         return
     }
 
     try {
         $AccessGroupACL = Get-Acl -Path "AD:\$($TheAccessGroup.DistinguishedName)" -ErrorAction Stop
     }
     catch {
         Write-Error "Failed to get the ACL of $($TheAccessGroup.DistinguishedName). The error was: $($Error[0])."
         return
     }
 
     $rootdse = Get-ADRootDSE
 
     $guidmap = @{}
     Get-ADObject -SearchBase ($rootdse.SchemaNamingContext) -LDAPFilter "(schemaidguid=*)" -Properties lDAPDisplayName,schemaIDGUID | % {$guidmap[$_.lDAPDisplayName]=[System.GUID]$_.schemaIDGUID}
 
     $extendedrightsmap = @{}
     Get-ADObject -SearchBase ($rootdse.ConfigurationNamingContext) -LDAPFilter "(&(objectclass=controlAccessRight)(rightsguid=*))" -Properties displayName,rightsGuid | % {$extendedrightsmap[$_.displayName]=[System.GUID]$_.rightsGuid}
 
     
     $AccessGroupACL.AddAccessRule((New-Object System.DirectoryServices.ActiveDirectoryAccessRule $OwnerGroupSid,$AccessRight,$AccessRule,$guidmap["$Property"])) | Out-Null
 
     try {
         Set-Acl -AclObject $AccessGroupACL -Path "AD:\$($TheAccessGroup.DistinguishedName)" -ErrorAction Stop
     }
     catch {
         Write-Error "Failed to set the ACL on $($TheAccessGroup.DistinguishedName). The error was: $($Error[0])."
         return
     }
 }
`

