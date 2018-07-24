---
Author: alphasun
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 3484
Published Date: 2012-06-28t13
Archived Date: 2012-12-17t23
---

# disable ad users - 

## Description

this script will disable all active directory user accounts that have not logged in within the number of days specified by the $numdays variable. all accounts that are disabled are logged in the “disabled-user-accounts.log” file created in the local directory. the formatting of the log file is very basic, but effective.

## Comments



## Usage



## TODO



## class

``

## Code

`#
 #
 $NumDays = 90
 $LogDir = ".\Disabled-User-Accounts.log"
 
 $currentDate = [System.DateTime]::Now
 $currentDateUtc = $currentDate.ToUniversalTime()
 $lltstamplimit = $currentDateUtc.AddDays(- $NumDays)
 $lltIntLimit = $lltstampLimit.ToFileTime()
 $adobjroot = [adsi]''
 $objstalesearcher = New-Object System.DirectoryServices.DirectorySearcher($adobjroot)
 $objstalesearcher.filter = "(&(objectCategory=person)(objectClass=user)(lastLogonTimeStamp<=" + $lltIntLimit + "))"
 $users = $objstalesearcher.findall()
 
 Write-Output `n`n"----------------------------------------" "ACCOUNTS OLDER THAN "$NumDays" DAYS" "PROCESSED ON:" $currentDate "----------------------------------------" `
 | Out-File $LogDir -append
 
 if ($users.Count -eq 0)
 {
        Write-Output "  No account needs to be disabled." | Out-File $LogDir -append
 }
 else
 {
        foreach ($user in $users)
        {
               [string]$adsPath = $user.Properties.adspath
               [string]$displayName = $user.Properties.displayname
               [string]$samAccountName = $user.Properties.samaccountname
               [string]$lastLogonInterval = $user.Properties.lastlogontimestamp
  
               $account=[ADSI]$adsPath
               $account.psbase.invokeset("AccountDisabled", "True")
               $account.setinfo()
  
               $lastLogon = [System.DateTime]::FromFileTime($lastLogonInterval)
              
               Write-Output "  Disabled user " $displayName" | Username: "$samAccountName" | Last Logon: "$lastLogon"`n" `
 			  | Out-File $LogDir -append
        }
 }
`

