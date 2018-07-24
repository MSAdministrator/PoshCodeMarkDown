---
Author: themoblin
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 4651
Published Date: 2014-11-26t16
Archived Date: 2014-02-09t11
---

# migrate-userssettings - 

## Description

migrates users settings roaming profiles and home directories to another server.

## Comments



## Usage



## TODO



## module

``

## Code

`#
 #
 Import-Module Activedirectory
 
 $oldserver = "hostname"
 
 $newserver = "hostname"
 
 $users = get-aduser -filter * -properties homedirectory,profilepath
 
 foreach ($user in $users) {
 	$name = $user.name
 	$DN = $user.distinguishedname
 	$profilepath = $user.profilepath
 	$homedirectory = $user.homedirectory
 
 	if ($profilepath -like "*$oldserver*")
 		{
 		Set-Aduser $DN -profilepath $profilepath.ToLower().replace($oldserver,$newserver)
 		}
 	else	{
 		Write-Host -foregroundcolor RED "User $name does not have roaming profile on $oldserver"
 		}
 
 	if ($homedirectory -like "*$oldserver*")
 		{
 		Set-Aduser $DN -homedirectory $homedirectory.ToLower().replace($oldserver,$newserver)
 		}
 	else	{
 		Write-Host -foregroundcolor RED "User $name does not have a Home directory on $oldserver"
 		}
 }
`

