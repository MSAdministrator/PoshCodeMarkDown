---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5137
Published Date: 
Archived Date: 2014-05-05t02
---

#  - 

## Description

i want to get this importing my $allmembers to my $list in sharepoint.

## Comments



## Usage



## TODO



## module

``

## Code

`#
 #
 import-module activedirectory
 
 "Active Directory Module Initiated"
 
 [string]$url = "http://myservername/Lists/SP_List_AD_Names/"
 $totalCount = 0
 
 $site = New-Object Microsoft.SharePoint.SPSite($url) 
 
 $web = $site.OpenWeb()
 
 $list = $web.Lists["SP_List_AD_Names"]
 
 foreach ($listItem in $list.Items)
 {
     $SubGroups = @()
 
     $AllMembers = @()
     
     $strGroupOwner = Get-ADGroup -identity $listItem.name -Properties ManagedBy | select managedby
     $strOwnerName = get-aduser -identity $strGroupOwner.managedby -properties samaccountname |select -ExpandProperty samaccountname
     $strGroupName = $listItem.Name
     "Group is named: " + $strGroupName
     "Group is owned by: " + $strOwnerName
     
     
     ForEach($Person in (Get-ADGroupMember $listItem.name -Recursive)){
         $totalCount ++
         $User = Get-ADUser $Person -Property description
     
         $AllMembers += New-Object PSObject -Property @{
     
             Name = $Person.Name
             Description = $User.Description
             NetworkID = $Person.SamAccountName
             Nested = $Null
             Group = $strGroupName
             Owner = $strOwnerName
     
                                                        }
 
         
                                                                     }
         
     $CurrentGroup = Get-ADGroupMember $ListItem.Name
     
 
 
 }
 
 
     "Current Group: " + $CurrentGroup
         
     Write-Host "Total Items = " $totalCount
`

