---
Author: dollarunderscore
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5386
Published Date: 2015-08-28t19
Archived Date: 2015-03-23t18
---

# get-adgrouplastused - 

## Description

this cmdlet could help with the process of finding out if a group is used in active directory by finding out when the members of the group logged on last time.

## Comments



## Usage



## TODO



## function

`get-adgrouplastused`

## Code

`#
 #
 #========================================================================
 #========================================================================
 
 function Get-ADGroupLastUsed
 {
 
     <#
     .SYNOPSIS
     This cmdlet checks when a member of the specified group logged on last time.
 
     .DESCRIPTION
     This cmdlet could help with the process of finding out if a group is used in Active Directory by finding out when the members of the group logged on last time.
     It returns what member in the group did the last logon and when that occured.
 
     This cmdlet requires the ActiveDirectory-module to be loaded for it to run.
 
     .EXAMPLE
     Get-ADGroupLastUsed -Identity "Domain Admins"
 
     Checks when a direct member of the group "Domain Admins" logged on last time.
 
     .EXAMPLE
     Get-ADGroupLastUsed -Identity "Domain Admins" -Recursive
 
     Checks when a member of the "Domain Admins" group or any child group logged in.
 
     .EXAMPLE
     Get-ADGroup -Filter "Name -like 'Admin*'" | Get-ADGroupLastUsed
 
     This will check all the groups named "Admin*" in the domain.
 
     .PARAMETER Identity
     Specify the SamAccountName, DistinguishedName, objectGUID or SID of the group to check. Supports pipeline input.
 
     .PARAMETER Server
     Specifies the Active Directory Domain Services instance to connect to.
 
     .PARAMETER Recursive
     When looking up the members, the cmdlet will get all members in the hierarchy of a group that do not contain child objects.
 
     #>
 
     [cmdletbinding()]
     param([Parameter(Mandatory=$True, ValueFromPipeline=$true, ValueFromPipelineByPropertyName=$true)]
           [Alias('SamAccountName','DistinguishedName','ObjectGUID','SID')]
           [string] $Identity,
           [string] $Server = $env:USERDNSDOMAIN,
           [switch] $Recursive)
 
     BEGIN { }
 
     PROCESS {
 
         $Parameters = @{ Identity = $Identity ; Recursive = $Recursive ; Server = $Server }
 
         $GroupLastUsed = Get-ADGroupMember @Parameters | Get-ADObject -Properties LastLogonTimeStamp | Sort-Object LastLogonTimeStamp -Descending | select -First 1 | select @{Name = 'ADGroup';Expression = { $Identity } }, @{Name = 'LastUsed';Expression = { (Get-Date ($_.LastLogonTimeStamp)).AddYears(1600) }}, @{Name = 'LastUsedBy';Expression = { $_.DistinguishedName } }
 
         Write-Output $GroupLastUsed
 
         Remove-Variable Parameters, GroupLastUsed -ErrorAction SilentlyContinu
     }
 
     END { }
 }
`

