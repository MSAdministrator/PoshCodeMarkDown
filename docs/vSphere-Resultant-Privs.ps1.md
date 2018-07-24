---
Author: carter shanklin
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 1520
Published Date: 
Archived Date: 2009-12-12t18
---

# vsphere resultant privs - 

## Description

given a user and a vsphere object, this code determins the specific privileges that user has on that object (a.k.a. the resultant privilege set).

## Comments



## Usage



## TODO



## function

`get-groups`

## Code

`#
 #
 Add-PSSnapin Quest.ActiveRoles* -ea SilentlyContinue
 
 function Get-Groups {
 	param($principal)
 
 	Write-Verbose "Checking principal $principal"
 	$groups = Get-QADUser $principal | Get-QADMemberOf
 
 	do {
 		$startLength = $groups.length
 		Write-Verbose ("Start length " + $startLength)
 		$groups += $groups | Get-QADMemberOf
 		$groups = $groups | Sort -Unique
 		$endLength = $groups.length
 		Write-Verbose ("End length " + $endLength)
 	} while ($endLength -ne $startLength)
 
 	Write-Output $groups
 }
 
 function Get-ResultantPrivileges {
 	param($principal, $viobject)
 
 	$account = (Get-QADUser $principal).NTAccountName
 	if ($account -eq $null) {
 		throw "$principal not found, please check your spelling."
 	}
 
 	$groups = Get-Groups -principal $account
 	$groupNames = $groups | Foreach { $_.Name }
 
 	$perms = $viobject | Get-VIPermission
 
 	$relevantPerms = $perms | Where {
 		(($_.IsGroup -eq $true) -and ($groupNames -contains $_.Principal)) -or
 		($_.Principal -eq $account)
 	}
 
 	$roleNames = $relevantPerms | Foreach { $_.Role } | Sort -Unique
 	Write-Verbose "Rolenames are $roleNames"
 	$roleObjects = Get-VIRole $roleNames
 	$roleCount = ($roleObjects | Measure-Object).Count
 
 	$roleObjects | Foreach { $_.PrivilegeList } | Group |
 		Where { $_.Count -eq $roleCount } |
 		Select Name
 }
 
`

