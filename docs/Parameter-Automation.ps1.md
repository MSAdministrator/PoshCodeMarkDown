---
Author: daniel sorlov
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: utf-8
License: cc0
PoshCode ID: 4851
Published Date: 2014-01-30t07
Archived Date: 2014-04-10t15
---

# parameter automation - 

## Description

pseudocode solution suggestion on sqlsetup parameter automation

## Comments



## Usage



## TODO



## function

`add-argument`

## Code

`#
 #
 
 function Add-Argument
 {
 	[CmdletBinding()]
 	[OutputType([string])]
 	PARAM(
 		[Parameter(Mandatory,Position=0,ValueFromPipelineByPropertyName)]
 		[ValidateNotNullOrEmpty]
 		[string]$Name,
 
 		[Parameter(Mandatory,Position=1,ValueFromPipelineByPropertyName)]
 		[ValidateNotNullOrEmpty]
 		[string]$Value
 	)
 
 	BEGIN
 	{
 		$result = "/{0}='{1}' -f $Name $Value		
 		Write-Output $result
 	}
 
 }
 
 ENDE - LÖSNING 1 - sätt värden i scriptet
 ########################################################################
 $processName = (Resolve-Path "setup.exe")
 $argumentList = @()
 
 $argumentList += Add-Argument "SQLSVCACCOUNT" "AnAccount"
 $argumentList += Add-Argument "SQLPASSWORD" "SecretStuff"
 $argumentList += Add-Argument "SQLCOLLATION" "SomeCollationId"
 
 Start-Process $processName -ArgumentList $argumentList
 
 ENDE - LÖSNING 2 - spara värden i csv fil enligt nedan:
 ########################################################################
 
 $processName = (Resolve-Path "setup.exe")
 Start-Process $processName -ArgumentList (Import-CSV .\MyARgumentFile.csv | Add-Argument)
`

