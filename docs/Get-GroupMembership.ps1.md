---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 583
Published Date: 
Archived Date: 2010-02-22t17
---

# get-groupmembership - 

## Description

two cmdlets for and from the active-directory uninitiated…

## Comments



## Usage



## TODO



## function

`get-distinguishedname`

## Code

`#
 #
 function Get-DistinguishedName { 
 Param($UserName)
    $ads = New-Object System.DirectoryServices.DirectorySearcher([ADSI]'')
    $ads.filter = "(&(objectClass=Person)(samAccountName=$UserName))"
    $s = $ads.FindOne()
    return $s.GetDirectoryEntry().DistinguishedName
 }
 
 function Get-GroupMembership {
 Param($DNName,[int]$RecurseLimit=-1)
 
    $groups = ([adsi]"LDAP://$DNName").MemberOf
    if ($groups -and $RecurseLimit) {
       Foreach ($gr in $groups) {
          $groups += @(Get-GroupMembership $gr -RecurseLimit:$($RecurseLimit-1) |
                     ? {$groups -notcontains $_})
       }
    }
    return $groups
 }
 
 #################################################################################
`

