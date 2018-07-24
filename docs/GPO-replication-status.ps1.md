---
Author: manuel toussaint
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 4810
Published Date: 2014-01-17t15
Archived Date: 2014-01-21t08
---

# gpo replication status - 

## Description

gpo replication status across domain controller.

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 
 param(
   [parameter(Mandatory = $True )][String]$GPOName
  )
 
 $DCList = (get-addomaincontroller -filter *).hostname 
 
 $colGPOVer = @()
 
 foreach ($DC in $DCList){
 
  $objGPOVers = New-Object System.Object
 
  $GPOObj = Get-GPO $GPOName -server $DC
 
  $UserVersion = [string]$GPOObj.User.DSVersion + ' (AD), ' + [string]$GPOObj.User.SysvolVersion + ' (sysvol)'
  $ComputerVersion = [string]$GPOObj.Computer.DSVersion + ' (AD), ' + [string]$GPOObj.Computer.SysvolVersion + ' (sysvol)'
 
  $objGPOVers | Add-Member -type noteproperty -name GPOName -value $GPOName
  $objGPOVers | Add-Member -type noteproperty -name DCName -value $DC
  $objGPOVers | Add-Member -type noteproperty -name UserVersion -value $UserVersion
  $objGPOVers | Add-Member -type noteproperty -name ComputerVersion -value $ComputerVersion
 
  $colGPOVer += $objGPOVers 
 }
 
 $colGPOVer | sort-object GPOName, DCName
`

