---
Author: alphasun
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5992
Published Date: 2015-08-26t18
Archived Date: 2015-09-10t17
---

# delete ad users - 

## Description

this script will delete all active directory user accounts that have not logged in within the number of days specified in the $numdays variable. the script only deletes the active directory user accounts, not any associated exchange mailboxes. the script also includes the delete-aduser function, which can be used separately from this script. all accounts that are deleted are logged in the “removed-user-accounts.log” file found in the local directory. the format of the log file is very basic, but effective.

## Comments



## Usage



## TODO



## function

`delete-aduser`

## Code

`#
 #
 function Delete-ADUser
 {
 	Param($userName = $(throw 'Enter a username to delete'))
 	$searcher = New-Object System.DirectoryServices.DirectorySearcher([ADSI]"","(&(objectcategory=user)(sAMAccountName=$userName))")
 	$user = $searcher.findone().GetDirectoryEntry()
 	$user.psbase.DeleteTree()
 }
 
 
 $NumDays = 90
 $LogDir = ".\Removed-User-Accounts.log"
 
 $currentDate = [System.DateTime]::Now
 $currentDateUtc = $currentDate.ToUniversalTime()
 $lltstamplimit = $currentDateUtc.AddDays(- $NumDays)
 $lltIntLimit = $lltstampLimit.ToFileTime()
 $adobjroot = [adsi]''
 $objstalesearcher = New-Object System.DirectoryServices.DirectorySearcher($adobjroot)
 $objstalesearcher.filter = "(&(objectCategory=person)(objectClass=user)(lastLogonTimeStamp<=" + $lltIntLimit + "))"
 $users = $objstalesearcher.findone()
 
 Write-Output `n`n"----------------------------------------" "ACCOUNTS OLDER THAN "$NumDays" DAYS" "PROCESSED ON:" $currentDate "----------------------------------------" `
 | Out-File $LogDir -append
 
 if ($users.Count -eq 0)
 {
        Write-Output "  No account needs to be removed." | Out-File $LogDir -append
 }
 else
 {
        foreach ($user in $users)
        {
               [string]$adsPath = $user.Properties.adspath
               [string]$displayName = $user.Properties.displayname
               [string]$samAccountName = $user.Properties.samaccountname
               [string]$lastLogonInterval = $user.Properties.lastlogontimestamp
  
               Delete-ADUser $samAccountName
  
               $lastLogon = [System.DateTime]::FromFileTime($lastLogonInterval)
              
               Write-Output "  Removed user " $displayName" | Username: "$samAccountName" | Last Logon: "$lastLogon"`n" `
 			  | Out-File $LogDir -append
        }
 }
`

