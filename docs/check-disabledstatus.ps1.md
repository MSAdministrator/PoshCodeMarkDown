---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 1066
Published Date: 
Archived Date: 2009-05-03t10
---

# check-disabledstatus - 

## Description

this script reads a list of usernames from a text file and outputs (to the screen) a comma-delimited list of usernames with a status value (ok, disabled or notfound).  this uses adsi.

## Comments



## Usage



## TODO



## class

``

## Code

`#
 #
 #
 
 if (!($args[0])) {
 	Write-Host "`nPlease specify a file containing usernames to check on the command line.`n" -ForegroundColor yellow
 	exit
 }
 
 $isdisabled = 0x02
 
 $searcher = new-object DirectoryServices.DirectorySearcher([ADSI]"")
 
 $userlist = Get-Content $args[0] | sort
 
 $i = 0
 
 foreach ($user in $userlist)
 {
 	$status  = "NOSUCHUSER"
 	$i++
 	
 	$pc = [int](($i / $userlist.count) * 100)
 	
 	Write-Progress -Activity "Checking users" -Status "$user..." -percentcomplete $pc
 	
 	$searcher.filter = "(&(objectClass=user)(sAMAccountName= $user))"
 	$founduser = $searcher.findOne()
 	
 	
 	if ($founduser.psbase.properties.useraccountcontrol) {
 			$status = "DISABLED"
 		} else {
 			$status = "OK"
 		}
 	}
 	Write-Host "$user, $status"
 }
`

