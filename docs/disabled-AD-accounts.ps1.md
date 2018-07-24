---
Author: cisco
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 3600
Published Date: 2012-08-29t03
Archived Date: 2012-09-05t09
---

# disabled ad accounts - 

## Description

retrieves from a list of samaccountname (txtfile) the accounts that are disabled in a forest.

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 $objForest = [System.DirectoryServices.ActiveDirectory.Forest]::GetCurrentForest()
 $DomainList = @($objForest.Domains | Select-Object Name)
 $Domains = $DomainList | foreach {$_.Name}
 
 $users = Get-Content U:\EMCU15FI3_USER.txt
 
 $total = $users.count
 "SAMaccountname;DisplayName;HomeDir;Domain" | Out-File -FilePath test.txt
 foreach($Domain in $Domains){
 	$i=0
 	foreach($user in $users) {
 		$b = Get-QADUser -SamAccountName $user -SizeLimit 0 -Disabled -DontUseDefaultIncludedProperties -IncludedProperties NTAccountName, DisplayName, HomeDirectory, userprincipalname -SerializeValues
 		$storing = $b.sAMAccountName + ";" + $b.displayName + ";" + $b.homeDirectory + ";" +$domain
 		if ($storing.StartsWith(";")) {
 			}
 			else{
 			$storing | Out-File -FilePath test.txt -append
 		}
 		$storing=$null
 		$b=$null
 		[System.GC]::Collect()
 		$i++;
 		Write-Progress -Activity "Searching disabled accounts in $domain" -Status "Progress:" -PercentComplete $($i*100/$total)
 	}
 }
`

