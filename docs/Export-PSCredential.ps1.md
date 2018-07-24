---
Author: hal rottenberg <hal@halr9000.com>
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: utf-8
License: cc0
PoshCode ID: 5899
Published Date: 2016-06-18t06
Archived Date: 2016-09-09t08
---

# export-pscredential - 

## Description

original filename

## Comments



## Usage



## TODO



## function

`export-pscredential`

## Code

`#
 #
 #
 #
 
 function Export-PSCredential {
 	param ( $Credential = (Get-Credential), $Path = "credentials.enc.xml" )
 
 	switch ( $Credential.GetType().Name ) {
 		PSCredential		{ continue }
 		String				{ $Credential = Get-Credential -credential $Credential }
 		default				{ Throw "You must specify a credential object to export to disk." }
 	}
 	
 	$export = "" | Select-Object Username, EncryptedPassword
 	
 	
 	$export.Username = $Credential.Username
 
 	$export.EncryptedPassword = $Credential.Password | ConvertFrom-SecureString
 
 	$export | Export-Clixml $Path
 	Write-Host -foregroundcolor Green "Credentials saved to: " -noNewLine
 
 	Get-Item $Path
 }
 
 function Import-PSCredential {
 	param ( $Path = "credentials.enc.xml" )
 
 	$import = Import-Clixml $Path 
 	
 	if ( !$import.UserName -or !$import.EncryptedPassword ) {
 		Throw "Input is not a valid ExportedPSCredential object, exiting."
 	}
 	$Username = $import.Username
 	
 	$SecurePass = $import.EncryptedPassword | ConvertTo-SecureString
 	
 	$Credential = New-Object System.Management.Automation.PSCredential $Username, $SecurePass
 	Write-Output $Credential
 }
`

