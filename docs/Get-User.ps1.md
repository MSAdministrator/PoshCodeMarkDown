---
Author: jonathan walz
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 735
Published Date: 
Archived Date: 2009-04-29t17
---

# get-user - 

## Description

this function is an attempt to duplicate the quest get-qaduser cmdlet without using any third party snap-ins. if you want to run it against a global catalog you simply need to replace ldap

## Comments



## Usage



## TODO



## function

`get-user`

## Code

`#
 #
 function Get-User($user)
 {
 	$dom = [System.DirectoryServices.ActiveDirectory.Domain]::GetCurrentDomain() 
 	$root = [ADSI] "LDAP://$($dom.Name)"
 	$searcher = New-Object System.DirectoryServices.DirectorySearcher $root
 	$searcher.filter = "(&(objectCategory=person)(objectClass=user)(cn=$user))"
 	$user = $searcher.FindOne()
 	[System.Collections.Arraylist]$names = $user.Properties.PropertyNames
 	[System.Collections.Arraylist]$props = $user.Properties.Values
 	$userobj = New-Object System.Object
 	for ($i = 0; $i -lt $names.Count)
 		{
 			$userobj | Add-Member -type NoteProperty -Name $($names[$i]) -Value $($props[$i])
 			$i++
 		}
 	$userobj.pwdlastset = [System.DateTime]::FromFileTime($userobj.pwdlastset)
 	$userobj.lastlogontimestamp = [System.DateTime]::FromFileTime($userobj.lastlogontimestamp)
 	return $userobj
 }
`

