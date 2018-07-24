---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 3517
Published Date: 2012-07-16t09
Archived Date: 2012-07-25t05
---

# compare-pathacl - 

## Description

compare the acls for two users for a folder tree

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 [CmdletBinding()]
 param(
 	[string]$Path = 'C:\',
 	[string]$User1 = "$Env:USERDOMAIN\$Env:UserName",
 	[string]$User2 = "BuiltIn\Administrators",
 	[switch]$recurse
 )
 foreach($fso in ls $path -recurse:$recurse) { 
 	$acl = @(get-acl $fso.FullName | select -expand Access | Where IdentityReference -in $user1,$user2) 
 	if($acl.Count -eq 1) { 
 		Write-Warning "Only $($acl[0].IdentityReference) has access to $($fso.FullName)"
 	} elseif($acl.Count -eq 2) { 
 		if(compare-object $acl[0] $acl[1] -Property FileSystemRights, AccessControlType) { 
 			Write-Warning "Different rights to $($fso.FullName)" 
 		}
 }
`

