---
Author: jan egil ring
Publisher: 
Copyright: 
Email: jan.egil.ring@powershell.no
Version: 1.0
Encoding: ascii
License: cc0
PoshCode ID: 4004
Published Date: 2013-03-08t07
Archived Date: 2016-06-10t06
---

# set-aduserrandompassword - set-aduserrandompassword.ps1

## Description

this script are used to set a random password for active directory users in a specified organizational unit. it stores the results in a csv-file.

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 ###########################################################################"
 #
 #
 #
 #
 #
 #
 ###########################################################################"
 
 
 $random = New-Object System.Random
 $CSV = @()
 Get-QADUser -SearchRoot "domain.local/MyUserOU" -SizeLimit 0 | ForEach-Object {
 $password = "pwd"+($random.Next(1000,9999))
 Set-QADUser $_ -UserPassword $password
 $exportdata = Get-QADUser $_ | Select-Object name,samaccountname,company,department
 Add-Member -InputObject $exportdata -MemberType NoteProperty -Name Password -Value $password
 $CSV += $exportdata
 }
 $CSV | Export-Csv -Path "C:\export\passwordlist.csv" -Encoding unicode -NoTypeInformation
`

