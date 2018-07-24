---
Author: st3v3o
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2071
Published Date: 2011-08-16t08
Archived Date: 2016-06-15t06
---

# update ad security group - 

## Description

update ad security group with users that have attribut x set.  this script does all updates on the pdc emulator.

## Comments



## Usage



## TODO



## module

`get-fsmoroles`

## Code

`#
 #
 if(@(get-module | where-object {$_.Name -eq "ActiveDirectory"} ).count -eq 0) {import-module ActiveDirectory}
 
 function Get-FSMORoles
 {
 Param (
   $Domain
   )
   
   $DomainDN = $Domain.defaultNamingContext
   
   $FSMO = @{}
   $PDC  = [adsi]("LDAP://"+ $DomainDN)
   $FSMO  = $FSMO + @{"PDC" = $PDC.fsmoroleowner}
   return $FSMO
 }
 $Role = (Get-FSMORoles ([adsi]("LDAP://RootDSE")))
 
 $PDC = $Role.PDC.ToString().split(",")[1]
 $PDC = $PDC.ToString().split("=")[1]
 
 $group="Test"
 
 $Users = Get-ADUser -Server $PDC -Filter {(department -eq "test") -and (objectclass -eq "user")}
 
 $checkGroup=Get-ADGroup -Server $PDC -Filter {(name -eq $group)}
 
 if($checkGroup -eq $null)
 	{echo "Group Doesn't Exist"; exit}
 
 $groupmembers = Get-ADGroupMember  "$group" -recursive -Server $PDC
 $gmembers = @()
 Foreach ($member in $groupmembers) {
 	$gmembers += $member.SamAccountName
 	}
 	
 Foreach ($User in $Users) {
 	If ($gmembers -notcontains $User.SamAccountName){Add-ADGroupMember -Server $PDC $group $User.SamAccountName }
 	}
`

