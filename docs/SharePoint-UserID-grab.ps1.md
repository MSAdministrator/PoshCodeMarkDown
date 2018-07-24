---
Author: craig pilkenton
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 740
Published Date: 
Archived Date: 2008-12-21t05
---

# sharepoint userid grab - 

## Description

use the moss (microsoft office sharepoint server) .net libraries to find sharepoint id of known user.

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 
 param(
 )
 
 Write-Host "Beginning Processing--`n"
 
 
 $strUserIDReturn = ""
 
 write-host "rqurdstrPath: $rqurdstrPath "
 write-host "rqurdstrListName: $rqurdstrListName "
 
 [void][System.reflection.Assembly]::LoadWithPartialName("Microsoft.SharePoint")
 $site = new-object Microsoft.SharePoint.SPSite($rqurdstrPath)
 $userlist = $site.RootWeb.Lists[$rqurdstrListName]
 
 write-host "site: $site "
 write-host "userlist: $userlist`n"
 
 if($strTargetUser -eq "") {
 	Write-Host "--Displaying all Portal Users"
 	foreach($strItem in $userlist.Items) {
 		[string]$strItem.ID + "--"+$strItem.Title
 	}
 }
 else {
 	write-host "--Targeting specific user to return ID"	
 	$intUserIDReturn = -1
 	
 	foreach($strItem in $userlist.Items) {
 		$intUserID=[int]$strItem.ID
 		$strUserName=$strItem.Title
 		if($strUserName -eq $strTargetUser) {
 			$intUserIDReturn = $intUserID
 			break;
 		}
 	}
 	$intUserIDReturn
 }
 
 $site.RootWeb.Dispose()
 $site.Dispose()
 
 
 
 Write-Host "`nEnd Processing--`n"
`

