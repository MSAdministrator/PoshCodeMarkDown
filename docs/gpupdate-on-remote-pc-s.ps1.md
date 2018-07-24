---
Author: afokkema
Publisher: 
Copyright: 
Email: 
Version: 1.0
Encoding: ascii
License: cc0
PoshCode ID: 2985
Published Date: 2012-10-05t09
Archived Date: 2012-01-14t07
---

# gpupdate on remote pc's - 

## Description

this script gets all pc’s or servers from a ou and runs gpupdate /force on these machines.

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 ###############################################################################
 #
 #
 ###############################################################################
 
 if ((Get-PSSnapin "Quest.ActiveRoles.ADManagement" `
             -ErrorAction SilentlyContinue) -eq $null) {
     Add-PSSnapin "Quest.ActiveRoles.ADManagement"
 }
 
 $url = "http://live.sysinternals.com/psexec.exe"
 $target = "c:\Tools\psexec.exe"
 $TempFile = "C:\Machines.txt"
 $Domain = Read-Host ("Enter the FQDN of the Domain")
 $OU = Read-Host ("Enter the name of the OU")
 
 if (test-path $target)
 {
 write-host "psexec.exe is already installed"
 } 
 else
 {
 write-host "psexec.exe doesn't exist"
 $wc = New-Object System.Net.WebClient
 $wc.DownloadFile($url, $target);
 write-host "psexec.exe is now installed"
 }
 
 
 $Servers = Get-QADComputer -SearchRoot $Domain/$OU | Select Name | out-file $TempFile -force
 
 (Get-Content $TempFile) | Foreach-Object {$_ -replace "Name                                                                           ", ""} | Set-Content $TempFile
 (Get-Content $TempFile) | Foreach-Object {$_ -replace "----                                                                           ", ""} | Set-Content $TempFile
 (Get-Content $TempFile) | Foreach-Object {$_ -replace "                                                                       ", ""} | Set-Content $TempFile
 (Get-Content $TempFile) | where {$_ -ne ""} >$TempFile
 
 $colComputers = Get-Content $TempFile
 Foreach ($strComputer in $colComputers){c:\Tools\psexec.exe \\$strComputer gpupdate.exe /target:computer /force}
 	
 RM $TempFile
`

