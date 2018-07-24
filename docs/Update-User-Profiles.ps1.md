---
Author: xspader
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5370
Published Date: 2015-08-15t00
Archived Date: 2015-06-23t00
---

# update user profiles - 

## Description

there was a script written by themoblin http

## Comments



## Usage



## TODO



## module

``

## Code

`#
 #
 $OldServer = 'OldServer'
 $NewServer = 'NewServer'
  
 Import-Module ActiveDirectory
 Get-ADUser -Filter "Enabled -eq 'true' -and HomeDirectory -like '*$OldServer*' -or ProfilePath -like '*$OldServer*'" -SearchBase "OU=xx,DC=xx,DC=xx,DC=xx" -Properties HomeDirectory, ProfilePath |
 ForEach-Object {
     if ($_.HomeDirectory -like "*$OldServer*") {
         Set-ADUser -Identity $_.DistinguishedName -HomeDirectory $($_.HomeDirectory -replace $OldServer, $NewServer)
     }
     if ($_.ProfilePath -like "*$OldServer*") {
         Set-ADUser -Identity $_.DistinguishedName -ProfilePath $($_.ProfilePath -replace $OldServer, $NewServer)
     }
 }
`

