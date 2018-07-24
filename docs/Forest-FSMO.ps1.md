---
Author: geoff guynn
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6031
Published Date: 2016-09-26t00
Archived Date: 2016-05-17t13
---

# forest fsmo - 

## Description

display the fsmo role holders of each domain in your forest.

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 $script:ProgramName = "Forest FSMO"
 $script:ProgramDate = "11 Dec 2013"
 $script:ProgramAuthor = "Geoffrey Guynn"
 $script:ProgramAuthorEmail = [System.Text.Encoding]::ASCII.GetString([System.Convert]::FromBase64String("Z2VvZmZyZXlAZ3V5bm4ub3Jn"))
 
 $Forest = [System.DirectoryServices.ActiveDirectory.Forest]::GetCurrentForest()
 $ChildDomains = $Forest.Domains
 $SchemaRole = $Forest.SchemaRoleOwner
 $NamingRole = $Forest.NamingRoleOwner
 
 $DomainObjects = @()
 
 $ChildDomains | % {
 
     $CurrentDomain = $_
     
     $objDomain = $CurrentDomain | Select Name, Forest, PDCRoleOwner, RidRoleOwner, InfrastructureRoleOwner
     $objDomain | Add-Member -MemberType NoteProperty -Name SchemaRole -Value $SchemaRole
     $objDomain | Add-Member -MemberType NoteProperty -Name NamingRole -Value $NamingRole
     
     $DomainObjects += $objDomain
 }
 
 $DomainObjects
`

