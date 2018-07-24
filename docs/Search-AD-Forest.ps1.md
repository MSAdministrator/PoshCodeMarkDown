---
Author: alphasun
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2933
Published Date: 2012-08-31t13
Archived Date: 2012-01-14t07
---

# search ad forest - 

## Description

this is essentially a snap-in for an existing script that leverages active directory. typically, you’ll be working with ad objects in your own domain; however, in some instances you may need to work with ad objects that are in a different domain within your forest. this code snippet allows the flexibility to drop in an existing domain-based script and either run it on all domains in the forest (no command line arguments) or a single domain in the forest that matches a command line argument placed into a where-object filter.

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 ########################################################################################
 ########################################################################################
 ########################################################################################
 ########################################################################################
 ##
 ##
 ##
 ##
 ##
 ##
 ##
 ##
 ##
 ##
 ##
 ########################################################################################
 
 [string]$arg = $Args[0]
 $objForest = [System.DirectoryServices.ActiveDirectory.Forest]::GetCurrentForest()
 $DomainList = @($objForest.Domains | Select-Object Name | Where-Object {$_ -like "*$arg*"})
 $Domains = $DomainLIst | foreach {$_.Name}
 
 foreach ($Domain in ($Domains))
 {
     Write-Host "Checking $Domain" -foregroundcolor Red
     Write-Host ""
 
     #############################
     #############################
 
 }
`

