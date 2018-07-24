---
Author: brian h madsen
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5577
Published Date: 2016-11-10t05
Archived Date: 2016-04-19t06
---

# site and lists + item - 

## Description

iterate site, list and items and show permissions – only adds items with unique permissions to report

## Comments



## Usage



## TODO



## function

`check-permission_spweb`

## Code

`#
 #
 $ParamObject = new-object PSObject
 
 $ParamObject | Add-Member Noteproperty OutputFile "C:\root\FarmPermissionReportByUser.csv"
 $ParamObject | Add-Member Noteproperty UserToCheck "es2\es2admin"
 $ParamObject | Add-Member Noteproperty StartUrl "http://goldfields"
 
 function Check-List-Item-Permission([string]$webUrl, [string]$loginToCheck) {
     [System.Reflection.Assembly]::LoadWithPartialName("Microsoft.SharePoint") > $null
     $mySite = new-object Microsoft.SharePoint.SPSite($webUrl)
     $myWeb = $mySite.openweb()
     $myWebUsers = $myWeb.Users
 
     foreach ($myWebUser in $myWebUsers) {        
         foreach ($myList in $myWeb.lists) {
             Write-Host " ------------ LIST: ", $myList.Title
             $listItems = $myList.Items
             foreach ($listItem in $listItems) {
                 Write-Host " -------------- ITEM: ", $listItem.Title, $listItem.Name
                 Write-Host " -------------- UNIQUE PERMISSIONS: ", $listItem.HasUniqueRoleAssignments
                 if ($listItem.HasUniqueRoleAssignments) {
                     "$($myweb.Url) `t Item `t $($listItem.Name)`t Permission `t $($mywebUser.LoginName)" | Out-File $ParamObject.OutputFile -Append
                 }
             }
         }
     }
 
 
 
 }
 
 function Check-Permission_SPWeb([string]$webURL, [string]$myLoginToCheck)
 {
 	[System.Reflection.Assembly]::LoadWithPartialName("Microsoft.SharePoint") > $null
 	$mysite = new-object Microsoft.SharePoint.SPSite($webURL)
 	$myweb = $mysite.openweb()
 	$mywebUsers = $myweb.Users
 
 	foreach($mywebUser in $mywebUsers)
 	{
 		if($mywebUser.LoginName -eq $myLoginToCheck)
 		{
 			$myPermissions = $myweb.Permissions
 			foreach($myPermission in $myPermissions)
 			{
 				if($mywebUser.ID -eq $myPermission.Member.ID)
 				{
 					Write-Host "      ----------------------------- "
 					Write-Host " ---- WEB Url: ", $webURL
 					Write-Host " ---- WEB Has Unique Permission:", $myweb.HasUniqueRoleAssignments
 					Write-Host " ---- User: ", $mywebUser.LoginName, " - Permissions: ", $myPermission.PermissionMask.ToString()
 
                     "$($myweb.Url) `t List `t $($myweb.Title)`t $($myPermission.PermissionMask.ToString()) `t $($mywebUser.LoginName)" | Out-File $ParamObject.OutputFile -Append
 
 					foreach ($role in $mywebUser.Roles)
 					{
 						if ($role.Type -ne [Microsoft.SharePoint.SPRoleType]::None)
 						{
 							Write-Host " ---- Role: ", $role.Type.ToString()
 						}
 					}
 					Write-Host "      ----------------------------- "
 				}
 			}
 			foreach($myList in $myweb.lists)
 			{
 				if($myList.HasUniqueRoleAssignments -eq $True)
 				{
 					$myListPermissions = $myList.Permissions
 					foreach($myListPermission in $myListPermissions)
 					{
 						if($mywebUser.ID -eq $myListPermission.Member.ID)
 						{
 							Write-Host "            ----------------------------- "
 							Write-Host " ---------- LIST NAME: ", $myList.Title
 							Write-Host " ---------- LIST Has Unique Permission:", $myList.HasUniqueRoleAssignments
 							Write-Host " ---------- User: ", $mywebUser.LoginName, " - Permissions: ", $myListPermission.PermissionMask.ToString()
 
                             "$($myweb.Url) `t List `t $($myweb.Title)`t $($myListPermission.PermissionMask.ToString()) `t $($mywebUser.LoginName)" | Out-File $ParamObject.OutputFile -Append
 
 							foreach ($roleAssignment in $myList.RoleAssignments)
 							{
 								if($mywebUser.ID -eq $roleAssignment.Member.ID)
 								{
 									foreach($mySPRoleDefinition in $roleAssignment.RoleDefinitionBindings)
 									{
 										if ($mySPRoleDefinition.Type -ne [Microsoft.SharePoint.SPRoleType]::None)
 										{
 											Write-Host " ---------- Role: ", $mySPRoleDefinition.Type.ToString()
 										}
 									}
 								}
 							}
 							Write-Host "            ----------------------------- "
                             
 						}
 					}
 				}
 			}
 		}
 	}
 	foreach ($subweb in $myweb.GetSubwebsForCurrentUser())
 	{
 		Check-Permission_SPWeb $subweb.Url $myLoginToCheck
     }
 	$myweb.Dispose()
 	$mysite.Dispose()
 }
 
 
 
 function ListUsers([string]$SiteCollectionURL, [string]$LoginToCheck)
 {
 	[System.Reflection.Assembly]::LoadWithPartialName("Microsoft.SharePoint") > $null
 	$site = new-object Microsoft.SharePoint.SPSite($SiteCollectionURL)
 	$web = $site.openweb()
 
 	$siteCollUsers = $web.SiteUsers
 
 	foreach($MyUser in $siteCollUsers)
 	{
 		if($MyUser.LoginName -eq $LoginToCheck)
 		{
 			Write-Host " ------------------------------------- "
 			Write-Host "Site Collection URL:", $SiteCollectionURL
 			if($MyUser.IsSiteAdmin -eq $true)
 			{
 				Write-Host "ADMIN: ", $MyUser.LoginName
                 "$($web.Url) `t Site `t $($web.Title)`t Administrator `t Administrator" | Out-File $ParamObject.OutputFile -Append
 			}
 			else
 			{
 				Write-Host "USER: ", $MyUser.LoginName
                 "$($web.Url) `t Site `t $($web.Title)`t User `t User" | Out-File $ParamObject.OutputFile -Append
 			}
             
 			    Check-Permission_SPWeb $web.Url $MyUser.LoginName
             #}
 			Write-Host " ------------------------------------- "
 		}
 	}
 	
 	$web.Dispose()
 	$site.Dispose()
 }
 
 function ListUsersForAllCollections([string]$WebAppURL, [string]$LoginToCheck)
 {
 
 	[System.Reflection.Assembly]::LoadWithPartialName("Microsoft.SharePoint") > $null
 
 	$Thesite = new-object Microsoft.SharePoint.SPSite($WebAppURL)
 	$oApp = $Thesite.WebApplication
 
 	foreach ($Sites in $oApp.Sites)
 	{
 		
         $mySubweb = $Sites.RootWeb
 		[string]$TempRelativeURL = $mySubweb.Url
 		ListUsers $TempRelativeURL $LoginToCheck
 
         Write-Host " --------------------"
         Check-List-Item-Permission $Sites.Url $LoginToCheck
     }
 }
 
 function StartProcess()
 {
 	[System.Diagnostics.Stopwatch] $sw;
 	$sw = New-Object System.Diagnostics.StopWatch
 	$sw.Start()
 	cls
 	
 	ListUsersForAllCollections $ParamObject.StartUrl $ParamObject.UserToCheck
 
 	$sw.Stop()
 
 	write-host "Login checked in Time: ", $sw.Elapsed.ToString()
 }
 
 "URL `t Site/List/Item `t Title `t PermissionType `t Permissions" | out-file $ParamObject.OutputFile
 
 
 
 StartProcess
`

