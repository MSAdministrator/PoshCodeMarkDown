---
Author: peter
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 1306
Published Date: 2010-09-03t15
Archived Date: 2014-06-01t17
---

# get moss user profile - 

## Description

quick functions to get either

## Comments



## Usage



## TODO



## function

`get-userprofile`

## Code

`#
 #
 function Get-UserProfile($accountName)
 {
 	[reflection.assembly]::LoadWithPartialName("Microsoft.SharePoint") | out-null
 	[reflection.assembly]::LoadWithPartialName("Microsoft.Office.Server") | out-null
 	
 	$upm =[Microsoft.Office.Server.UserProfiles.UserProfileManager](
 		[Microsoft.Office.Server.ServerContext]::Default)
 		
 	$upm.GetUserProfile($accountName)
 }
 
 function Get-UserProfileData($profile)
 {
 	$propsToDisplay = $upm.Properties | ? { $_.IsSystem -eq $false -and $_.IsSection -eq $false }
 	
 	$o = new-object PSObject
 	$propsToDisplay | foreach {
 		add-member -inp $o -membertype NoteProperty -name $_.Name -value $profile[$_.Name].Value
 	}
 	
 	$o
 }
 
 
 
 
 #
 #
 
 
 
 #
`

