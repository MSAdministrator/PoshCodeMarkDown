---
Author: jan egil ring
Publisher: 
Copyright: 
Email: jan.egil.ring@powershell.no
Version: 1.0/2.0
Encoding: ascii
License: cc0
PoshCode ID: 4131
Published Date: 2015-04-25t12
Archived Date: 2015-08-28t18
---

# add-networkprinter.ps1 - add-networkprinter.ps1

## Description

windows powershell script to map a network printer based on active directory group membership.

## Comments



## Usage



## TODO



## script

`get-groupmembership`

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
 
 $strName = $env:username
 
 function get-GroupMembership($DNName,$cGroup){
 	
 	$strFilter = "(&(objectCategory=User)(samAccountName=$strName))"
 
 	$objSearcher = New-Object System.DirectoryServices.DirectorySearcher
 	$objSearcher.Filter = $strFilter
 
 	$objPath = $objSearcher.FindOne()
 	$objUser = $objPath.GetDirectoryEntry()
 	$DN = $objUser.distinguishedName
 		
 	$strGrpFilter = "(&(objectCategory=group)(name=$cGroup))"
 	$objGrpSearcher = New-Object System.DirectoryServices.DirectorySearcher
 	$objGrpSearcher.Filter = $strGrpFilter
 	
 	$objGrpPath = $objGrpSearcher.FindOne()
 	
 	If (!($objGrpPath -eq $Null)){
 		
 		$objGrp = $objGrpPath.GetDirectoryEntry()
 		
 		$grpDN = $objGrp.distinguishedName
 		$ADVal = [ADSI]"LDAP://$DN"
 	
 		if ($ADVal.memberOf.Value -eq $grpDN){
 			$returnVal = 1
 			return $returnVal = 1
 		}else{
 			$returnVal = 0
 			return $returnVal = 0
 	
 		}
 	
 	}else{
 			$returnVal = 0
 			return $returnVal = 0
 	
 	}
 		
 }
 
 $result = get-groupMembership $strName "Printer_group_01"
 if ($result -eq '1') {
 Invoke-Expression 'rundll32 printui.dll,PrintUIEntry /in /q /n"\\print-server\printer-share"'
 }
`

