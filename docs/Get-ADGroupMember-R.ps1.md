---
Author: error_success
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 4181
Published Date: 2014-05-24t03
Archived Date: 2017-05-16t03
---

# get-adgroupmember -r - 

## Description

this is how i would go about recursively getting ad group members.

## Comments



## Usage



## TODO



## function

`get-membersfromad`

## Code

`#
 #
 function Get-MembersFromAD{
     [cmdletbinding()]
     Param(
         [Parameter(Mandatory=$true)]
         [string]$DistinguishedGroupName
     )
 
     Write-Verbose "Getting `"$DistinguishedGroupName`""
     $group = [adsi]"LDAP://$DistinguishedGroupName"
 
     Write-Verbose "Getting members ..."
     foreach($DN in $group.member){        
         $member = [adsi]"LDAP://$DN"
         if($member.objectClass -contains 'group'){
             Write-Verbose "RECURSIVE"
             $peeps += @(Get-MembersFromAD $member.distinguishedName -Verbose)
         }
         else{
             $peeps += @($member.sAMAccountName)
         }
     }
 
     return $peeps
 }
 
 Get-MembersFromAD 'CN=Group Name,OU=Groups,OU=Practice,OU=Location,DC=company,DC=com' -Verbose
`

