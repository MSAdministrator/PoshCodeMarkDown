---
Author: alphasun
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6406
Published Date: 2016-06-25t16
Archived Date: 2016-06-29t04
---

# list ad users csv - 

## Description

this script will list all user objects in the current active directory domain. the data gathered includes display name, username, last logon date, and disabled status. all data is exported to a csv file.

## Comments



## Usage



## TODO



## class

``

## Code

`#
 #
 $NumDays = 0
 $LogDir = ".\User-Accounts.csv"
 
 $currentDate = [System.DateTime]::Now
 $currentDateUtc = $currentDate.ToUniversalTime()
 $lltstamplimit = $currentDateUtc.AddDays(- $NumDays)
 $lltIntLimit = $lltstampLimit.ToFileTime()
 $adobjroot = [adsi]''
 $objstalesearcher = New-Object System.DirectoryServices.DirectorySearcher($adobjroot)
 $objstalesearcher.filter = "(&(objectCategory=person)(objectClass=user)(lastLogonTimeStamp<=" + $lltIntLimit + "))"
 
 $users = $objstalesearcher.findall() | select `
 @{e={$_.properties.cn};n='Display Name'},`
 @{e={$_.properties.samaccountname};n='Username'},`
 @{e={[datetime]::FromFileTimeUtc([int64]$_.properties.lastlogontimestamp[0])};n='Last Logon'},`
 @{e={[string]$adspath=$_.properties.adspath;$account=[ADSI]$adspath;$account.psbase.invokeget('AccountDisabled')};n='Account Is Disabled'}
 
 $users | Export-CSV -NoType $LogDir
`

