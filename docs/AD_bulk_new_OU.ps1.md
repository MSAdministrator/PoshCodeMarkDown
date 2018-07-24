---
Author: chriskenis
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6346
Published Date: 2016-05-17t08
Archived Date: 2016-05-19t19
---

# ad_bulk_new_ou - 

## Description

active directory, bulk create ou’s with defined sub ou’s

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 param(
 $searchBase = "OU=Organisation,DC=uza,DC=local",
 $NewOUs = @(Import-csv -Path "d:\projects\AD\departments.csv" -Delimiter ";"),
 $SubOUs = @("Computers","Users"),
 [switch]$ProtectOU
 )
 $Protect = $false
 If ($ProtectOU){$Protect = $true}
 
 foreach ($NewOU in $NewOUs){
 New-ADOrganizationalUnit -Name $NewOU.name -Description $NewOU.description -City "Antwerp" -Country "BE" -ManagedBy $NewOU.manager -State "Antwerp" -Path $searchBase -ProtectedFromAccidentalDeletion $Protect
 $SubOUPath = "OU=" + $Newou.Name + "," + $searchBase
 foreach ($SubOU in $SubOUs){
 New-ADOrganizationalUnit -Name $SubOU -Path $SubOUPath -ProtectedFromAccidentalDeletion $Protect
 }
 }
 
`

