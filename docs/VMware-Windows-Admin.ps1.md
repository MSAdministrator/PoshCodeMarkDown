---
Author: clint jones
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 3441
Published Date: 2013-05-30t14
Archived Date: 2016-03-05t23
---

# vmware / windows admin - 

## Description

use to get esxi host versions

## Comments



## Usage



## TODO



## module

``

## Code

`#
 #
 #========================================================================
 #========================================================================
 
 Add-PSSnapin "Vmware.VimAutomation.Core"
 
 $log = "C:\Scripts\VMware\Logs\hostversions.txt"
 
 $pathverif = Test-Path -Path c:\scripts\vmware\logs
 switch ($pathverif)
     {
         True    {}
         False   {New-Item "c:\scripts\vmware\logs" -ItemType directory}
         default {Write-Host "An error has occured when checking the file structure"}
     }
 
 $viserver = Read-Host "Enter VMware server:"
 $creds = Get-Credential
 Connect-ViServer -Server $viserver -Credential $creds
 
 Get-VMHost | Select-Object Name, Version | Format-Table -AutoSize | Out-File -FilePath $log -Append
`

