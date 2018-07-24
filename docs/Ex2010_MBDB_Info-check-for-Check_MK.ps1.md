---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 1.0
Encoding: ascii
License: cc0
PoshCode ID: 4110
Published Date: 
Archived Date: 2013-05-09t09
---

#  - 

## Description

ex2010_mbdb_info check for check_mk

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 If ([intptr]::size -eq 4) {
 	$powerShellDir = $powershelldir = "$env:Windir\Sysnative\WindowsPowerShell\V1.0\"
 	$dir = "& `"$env:ProgramFiles\Check_MK\plugins\Ex2010_MBDB_Info.ps1`""
 	$bytes = [Text.Encoding]::Unicode.GetBytes($dir)
 	$encodedCommand = [Convert]::ToBase64String($bytes)
 	Invoke-Command -command { & $powershelldir\powershell.exe -EncodedCommand $encodedCommand }
 }
 Else {
 	Write-Host '<<<Ex2010_MBDB_Info>>>' 
 	Add-PSSnapin Microsoft.Exchange.Management.PowerShell.E2010 
 	$DBS = Get-MailboxDatabase -Status -Server (hostname) 
 	
 	$DBS | % { '{0,-10} {1,-20} {2, -6}' -f $_.name, ($_.Databasesize -replace ',', ''-replace '^.*\((.*) bytes\)', '$1'), (Get-MailboxStatistics -Database $_.name|where { $_.DisconnectReason -ne 'SoftDeleted' }).count };
 }
`

