---
Author: chris brown
Publisher: 
Copyright: 
Email: 
Version: 1.0
Encoding: ascii
License: cc0
PoshCode ID: 5651
Published Date: 2014-12-19t00
Archived Date: 2014-12-27t09
---

# get-quizzlefact - 

## Description

this script retrieves one or more facts from the quizzle.me api

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 <#
 .SYNOPSIS
     This script retrieves one or more facts from the Quizzle.me API
 .DESCRIPTION
     This script retrieves one or more facts from the Quizzle.me API
 
     IMPORTANT NOTE:
     This script cannot be held accountable for the amount of truth included
     in retrieved facts
 .NOTES
     File Name  : Get-QuizzleFact.ps1
     Author     : Chris Brown (chris@chrisbrown.id.au)
     Date       : 19/12/2014 11:15
     Version    : 1.0
 	
 	Revisions:
 	v1.0: 19/12/2014 Initial script
 		
 .PARAMETER Count
     The number of facts to return. Default: 1
 .EXAMPLE
     .\Get-QuizzleFact.ps1
 .EXAMPLE
     .\Get-QuizzleFact.ps1 -count 3
 
 #>
 
 [CmdletBinding()]
 Param(
       [Parameter(Mandatory=$false,HelpMessage="Enter the number of facts to return. Default: 1")][int]$Count = 1
       )
 
 ############################################################
 ############################################################
 
 $apiurl = "http://quizzle.me/api/facts.json"
 
 ############################################################
 ############################################################
 function GetFactAsString() {
     (Invoke-WebRequest $apiurl).Content | ConvertFrom-Json | select -ExpandProperty rand_record | Select -ExpandProperty fact_string
 }
 
 ############################################################
 ############################################################
 
 for($i=1; $i -le $count; $i++){
     GetFactAsString
 }
`

