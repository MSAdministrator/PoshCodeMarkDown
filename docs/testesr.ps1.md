---
Author: ccorin
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6612
Published Date: 2016-11-06t21
Archived Date: 2016-11-09t11
---

# testesr - 

## Description

#stored credential code

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 $AdminName = Read-Host "Enter your Admin AD username"
 $CredsFile = "C:\$AdminName-PowershellCreds.txt"
 $FileExists = Test-Path $CredsFile
 if  ($FileExists -eq $false) {
 	Write-Host 'Credential file not found. Enter your password:' -ForegroundColor Red
 	Read-Host -AsSecureString | ConvertFrom-SecureString | Out-File $CredsFile
 	$password = get-content $CredsFile | convertto-securestring
 	$Cred = new-object -typename System.Management.Automation.PSCredential -argumentlist domain\$AdminName,$password}
 else 
 	{Write-Host 'Using your stored credential file' -ForegroundColor Green
 	$password = get-content $CredsFile | convertto-securestring
 	$Cred = new-object -typename System.Management.Automation.PSCredential -argumentlist domain\$AdminName,$password}
 sleep 2
 
 Write-Host 'Connecting to Active Directory'
 Connect-QADService -Service 'gbay-ad01' -Credential $Cred -ErrorAction Stop | out-Null
 $Session = New-PSSession -ConfigurationName Microsoft.Exchange -ConnectionUri http://server.fqdn.com/PowerShell/ -Credential $Cred -Authentication Kerberos -ErrorAction SilentlyContinue
 Import-PSSession $Session -ErrorAction SilentlyContinue -AllowClobber
 if(!$?)
 	{write-host "Failed importing the exchange pssession, exiting!"
 	exit}
`

