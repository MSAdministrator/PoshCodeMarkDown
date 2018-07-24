---
Author: crazydave
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 4520
Published Date: 2014-10-15t17
Archived Date: 2017-03-18t05
---

# get-wifiprofiles - 

## Description

uses netsh command to get all the computer’s wifi profiles (including clear text passwords)

## Comments



## Usage



## TODO



## function

`get-wifiprofiles`

## Code

`#
 #
 function Get-WifiProfiles {
 $tmpFolder = Join-Path $env:temp ([Guid]::NewGuid().ToString().Replace('-',''))
 mkdir $tmpFolder | Out-Null
 
 netsh wlan export profile key=clear folder=$tmpFolder | Out-Null
 
 gci $tmpFolder\*.xml | % { 
     $data = [xml] (gc $_)
     New-Object PSObject -Property @{
         Name = $data.WLANProfile.name
         SSID = $data.WLANProfile.SSIDConfig.SSID.name
         ConnType = $data.WLANProfile.connectionType
         ConnMode = $data.WLANProfile.connectionMode
         AuthType = $data.WLANProfile.MSM.security.authEncryption.authentication
         Encryption = $data.WLANProfile.MSM.security.authEncryption.encryption
         KeyType = $data.WLANProfile.MSM.security.sharedKey.keyType
         Key = $data.WLANProfile.MSM.security.sharedKey.keyMaterial
     }
 }
 
 
 ri -Force -Recurse -Path $tmpFolder
 }
`

