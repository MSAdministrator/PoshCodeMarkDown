---
Author: sean kearney
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2121
Published Date: 2012-09-06t12
Archived Date: 2012-06-15t18
---

# get-remoteapps - 

## Description

find an application on remote list of computers.

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 #
 
 
 $COMPUTERS=IMPORT-CSV C:\Powershell\Computerlist.csv
 
 FOREACH ($PC in $COMPUTERS) {
 $computername=$PC.NameOfComputer
 
 $Branch='LocalMachine'
 
 $SubBranch="SOFTWARE\\Microsoft\\Windows\\CurrentVersion\\Uninstall"
 
 $registry=[microsoft.win32.registrykey]::OpenRemoteBaseKey('Localmachine',$computername)
 $registrykey=$registry.OpenSubKey($Subbranch)
 $SubKeys=$registrykey.GetSubKeyNames()
 
 
 Foreach ($key in $subkeys)
 {
     $exactkey=$key
     $NewSubKey=$SubBranch+"\\"+$exactkey
     $ReadUninstall=$registry.OpenSubKey($NewSubKey)
     $Value=$ReadUninstall.GetValue("DisplayName")
     Write-Host $computername, $Value
  
 }
`

