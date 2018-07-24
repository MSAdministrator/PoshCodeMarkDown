---
Author: z3r0c00l12
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 3750
Published Date: 2012-11-06t11
Archived Date: 2012-11-11t22
---

# get-recursemember - 

## Description

a function for recursively getting a list of unique users that are members of a domain group.

## Comments



## Usage



## TODO



## function

`get-recursemember`

## Code

`#
 #
 function Get-RecurseMember {
 <#
 .Synopsis
   Does a recursive search for unique users that are members of an AD group.
 
 .Description
   Recursively gets a list of unique users that are members of the specified 
   group, expanding any groups that are members out into their member users.
 
   Note: Requires the Quest AD Cmdlets
         http://www.quest.com/powershell/activeroles-server.aspx
 
 .Parameter group
   The name of the group.
 
 .Example
 PS> Get-RecurseMember 'My Domain Group'
 
 .Notes
   NAME:      Get-RecurseMember
   AUTHOR:    tojo2000
 #>
   param([Parameter(Position = 0,
                    Mandatory = $true)]
         [string]$group)
   $users = @{}
   
   try {
     $members = Get-QADGroupMember $group
   } catch [ArgumentException] {
     Write-Host "`n`n'$group' not found!`n"
     return $null
   }
   
   foreach ($member in $members) {
     if ($member.Type -eq 'user') {
       $users.$($member.Name.ToLower()) = $member
     } elseif ($member.Type -eq 'group') {
       foreach ($user in (Get-RecurseMember $member.Name)) {
         $users.$($user.Name.ToLower()) = $user
       }
     }
   }
   
   foreach ($user in $users.Keys | sort) {
     Write-Output $users.$user
   }
 }
`

