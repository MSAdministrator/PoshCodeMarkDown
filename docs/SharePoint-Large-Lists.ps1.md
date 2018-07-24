---
Author: peter
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 3110
Published Date: 2012-12-20t13
Archived Date: 2016-08-26t03
---

# sharepoint large lists - 

## Description

provides details for every large list in the farm. as it is utilitarian, this script does not provide periodic status updates, though it could be programmed to do so.

## Comments



## Usage



## TODO



## function

`is-admin`

## Code

`#
 #
 [reflection.assembly]::loadwithpartialname("Microsoft.SharePoint")
 $cs = [microsoft.sharepoint.administration.spwebservice]::ContentService
 $global:largeListThreshhold = 2000
 
 function Is-Admin([Microsoft.SharePoint.SPRoleAssignment]$roleAssignment)
 {
 	return (($roleAssignment.roledefinitionbindings | where { ($_.BasePermissions.ToString() -match "ManageLists|ManageWeb|FullMask") }) -ne $null)
 }
 
 function Write-ListDetails($list,$web,$site)
 {
 	$fields = @()
 	$fields += $list.Title
 	$fields += $list.RootFolder
 	$fields += ($list.ContentTypes | select -first 1).Name
 	$fields += ($list.ContentTypes | select -first 1).Id.ToString()
 	$fields += ($list.ContentTypes | select -first 1).Id.Parent.ToString()
 	$fields += $list.Items.NumberOfFields
 	$fields += $list.Items.Count
 	$fields += $list.Created
 	$fields += $list.LastItemModifiedDate
 	
 	$listAdmins = @()
 	$listAdmins += $list.RoleAssignments | where { Is-Admin $_ }
          if ($list.RoleAssignments.Count -gt 0 -and $listAdmins.Count -gt 0)
 	{
 		$fields += $listAdmins[0].Member.Name
 		$fields += $listAdmins[0].Member.Email
 		$fields += [string]::Join("; ", @($listAdmins | % { $_.Member.ToString() } ))
 	} else {
 		$fields += ""
 		$fields += ""
 		$fields += ""
 	}
 	$fields += $web.Url
 	$fields += $web.Title
 	$fields += $site.Url
 	
 	[string]::Join("`t", $fields)
 }
 
 function Enumerate-LargeListsInSite($siteCollection)
 {
 	foreach ($web in $siteCollection.AllWebs) {
 		$web.Lists | where { $_.Items.Count -gt $global:largeListThreshhold } | foreach { Write-ListDetails -list $_ -web $web -site $siteCollection }
 		
 		$web.Dispose()
 	}
 }
 
 function List-FieldNames
 {
 	$fieldNames = @()
 	$fieldNames += "ListTitle"
 	$fieldNames += "ListRootFolderURL"
 	$fieldNames += "ContentType1Name"
 	$fieldNames += "ContentType1ID"
 	$fieldNames += "ContentType1ParentID"
 	$fieldNames += "NumberOfFields"
 	$fieldNames += "Items"
 	$fieldNames += "CreatedDate"
 	$fieldNames += "LastItemModifiedDate"
 	$fieldNames += "ListAdmin1Name"
 	$fieldNames += "ListAdmin1Email"
 	$fieldNames += "AllListAdmins"
 	$fieldNames += "WebURL"
 	$fieldNames += "WebTitle"
 	$fieldNames += "SiteURL"
 	
 	return [string]::Join("`t", $fieldNames)
 }
 
 function Enumerate-LargeLists
 {
 	List-FieldNames
 		
 	$webApplications = $cs.WebApplications | foreach { $_ }
 	foreach ($webApplication in $webApplications) {
 		$webApplication.Sites | foreach {
 			Enumerate-LargeListsInSite -siteCollection $_
 			
 			$_.Dispose()
 		}
 	}
 }
 
 
`

