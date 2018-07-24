---
Author: redyey
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5691
Published Date: 2015-01-14t17
Archived Date: 2015-01-31t21
---

# password functions - 

## Description

new-passwordfile

## Comments



## Usage



## TODO



## function

`new-passwordfile`

## Code

`#
 #
 function New-PasswordFile
 {
 	param (
 		[parameter(Mandatory = $True)]
 		[String]$PasswordFile
 	)
 	
 	Read-Host -AsSecureString "Enter a password" | ConvertFrom-SecureString | Out-File $PasswordFile -ErrorAction Stop
 }
 
 function Get-PasswordFromEncryptedFile
 {
 	param (
 		[parameter(Mandatory = $True)]
 		[string]$PasswordFile
 	)
 	
 	if (!(Test-Path $PasswordFile))
 	{
 		throw "Nonexistent Password file"
 	}
 	
 	else
 	{
 		$EncryptedPass = Get-Content $PasswordFile | ConvertTo-SecureString
 		$WncryptedStr = [System.Runtime.InteropServices.Marshal]::SecureStringToBSTR($EncryptedPass)
 	}
 	
 }
 
 if (!(Test-Path C:\password.txt))
 {
 	New-PasswordFile -PasswordFile "C:\password.txt"
 }
 
 $Pass = Get-PasswordFromEncryptedFile -PasswordFile "C:\password.txt"
`

