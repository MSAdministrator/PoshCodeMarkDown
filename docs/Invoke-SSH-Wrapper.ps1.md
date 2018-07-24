---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 646
Published Date: 
Archived Date: 2009-04-22t16
---

# invoke-ssh wrapper - 

## Description

this function is a wrapper for the invoke-ssh cmdlet which is included in the netcmdlets package (http

## Comments



## Usage



## TODO



## function

`invoke-ssh`

## Code

`#
 #
 function Invoke-SSH {
 	param (
 		[string]$Server = "$(throw 'Server is a mandatory parameter.')",
 		$Credential = "$(throw 'Credential is a mandatory parameter.')",
 		[switch]$Echo = $true,
 		[string]$Command = "$(throw 'Command is a mandatory parameter.')"
 	)
 	$ShellObject = Netcmdlets\Invoke-SSH -Server $Server -Credential $Credential -Force -Command $Command
 	$Output = $ShellObject | ForEach-Object { ($_.Text).split("`n") }
 	if ( $Echo ) {
 		$UserName = $Credential.UserName -replace "\\"
 	}
 	Write-Output $Output
 }
`

