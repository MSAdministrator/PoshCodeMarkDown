---
Author: lee holmes
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2145
Published Date: 2011-09-09t21
Archived Date: 2016-05-17t11
---

# get-aclmisconfiguration. - 

## Description

from windows powershell cookbook (o’reilly) by lee holmes

## Comments



## Usage



## TODO



## function

``

## Code

`#
 #
 ##############################################################################
 ##
 ##
 ##
 ##############################################################################
 
 <#
 
 .SYNOPSIS
 
 Demonstration of functionality exposed by the Get-Acl cmdlet. This script
 goes through all access rules in all files in the current directory, and
 ensures that the Administrator group has full control of that file.
 
 #>
 
 Set-StrictMode -Version Latest
 
 foreach($file in Get-ChildItem)
 {
     $acl = Get-Acl $file
     if(-not $acl)
     {
         continue
     }
 
     $foundAdministratorAcl = $false
 
     foreach($accessRule in $acl.Access)
     {
         if(($accessRule.IdentityReference -like "*Administrator*") -and
             ($accessRule.FileSystemRights -eq "FullControl"))
         {
             $foundAdministratorAcl = $true
         }
     }
 
     if(-not $foundAdministratorAcl)
     {
         "Found possible ACL Misconfiguration: $file"
     }
 }
`

