---
Author: dalibor zacek
Publisher: 
Copyright: 
Email: 
Version: 1.0
Encoding: ascii
License: cc0
PoshCode ID: 4689
Published Date: 2013-12-11t06
Archived Date: 2013-12-13t12
---

# copy-adgroup - 

## Description

creates copy of existing group in active direcory domain with members and other properties.

## Comments



## Usage



## TODO



## function

`copy-adgroup`

## Code

`#
 #
 
 function Copy-ADGroup {
 <#
 .SYNOPSIS
     Creates copy of existing group in Active Direcory domain with members and other properties.
 .DESCRIPTION
     Copies group scope, type, members, description and notes.
 .PARAMETER Source
     Name of source group.
 .PARAMETER Target
     Name of target group that will be created.
 .PARAMETER Path
     Specifies the X.500 path of the Organizational Unit (OU) or container where the new group is created.
 .NOTES
     Author: Dalibor Zacek
     Version: 1.0
     Requirements: Active Directory module provider 
         http://technet.microsoft.com/en-us/library/ee617195.aspx
 
 .EXAMPLE
     Copy-ADGroup -Source "Contoso Sales" -Target "Contoso Marketing"
 .EXAMPLE
     Copy-ADGroup "Sales" "Finance" -Path "OU=Groups,DC=Contoso,DC=com"
 #>
 [CmdletBinding()]
 param (
     [parameter(Mandatory=$true,Position=1)]
     [string]$Source,
 
     [parameter(Mandatory=$true,Position=2)]
     [string]$Target,
 
     [parameter(Mandatory=$false,Position=3)]
     [string]$Path
 )
 Begin {
         Try {  
             If ( -not (Get-Module -Name "ActiveDirectory")) {  
                 Import-Module -Name ActiveDirectory  
             } 
         } Catch {  
             Write-Warning "Failed to Import REQUIRED Active Directory Module...exiting script! `nhttp://technet.microsoft.com/en-us/library/ee617195.aspx" 
             Break  
         }
 }
 Process {
     $SourceGroup = Get-ADGroup -Identity $Source -Properties Description,CN,info,Members
     $SourceMembers = $SourceGroup.Members
     $SourceNotes = $SourceGroup.info
     
     if (-not $Path) {
         $Path = $SourceGroup.DistinguishedName.Substring(4+($SourceGroup.CN.Length),$SourceGroup.DistinguishedName.Length-(4+($SourceGroup.CN.Length)))
         Write-Verbose "Path: $Path"
     }
     Write-Verbose "Type: $($SourceGroup.GroupCategory)"
     Write-Verbose "Scope: $($SourceGroup.GroupScope)"
     Write-Verbose "Description: $($SourceGroup.Description)"
     Write-Verbose "Notes: $SourceNotes"
     if ($SourceNotes) {
         New-ADGroup -Name $Target -Description $SourceGroup.Description -GroupCategory $SourceGroup.GroupCategory -GroupScope $SourceGroup.GroupScope -Path $Path -OtherAttributes @{info=($SourceGroup.info)}
     } else {
         New-ADGroup -Name $Target -Description $SourceGroup.Description -GroupCategory $SourceGroup.GroupCategory -GroupScope $SourceGroup.GroupScope -Path $Path
     }
     foreach ($Member in $SourceMembers) { 
         Add-ADGroupMember -Identity $Target -Members $Member 
     }
 }
 }
`

