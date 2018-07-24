---
Author: lee holmes
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2162
Published Date: 2011-09-09t21
Archived Date: 2016-03-18t21
---

# get-remoteregistrychildi - 

## Description

from windows powershell cookbook (o’reilly) by lee holmes

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 ##############################################################################
 ##
 ##
 ##
 ##############################################################################
 
 <#
 
 .SYNOPSIS
 
 Get the list of subkeys below a given key on a remote computer.
 
 .EXAMPLE
 
 Get-RemoteRegistryChildItem LEE-DESK HKLM:\Software
 
 #>
 
 param(
     [Parameter(Mandatory = $true)]
     $ComputerName,
 
     [Parameter(Mandatory = $true)]
     $Path
 )
 
 Set-StrictMode -Version Latest
 
 if($path -match "^HKLM:\\(.*)")
 {
     $baseKey = [Microsoft.Win32.RegistryKey]::OpenRemoteBaseKey(
         "LocalMachine", $computername)
 }
 elseif($path -match "^HKCU:\\(.*)")
 {
     $baseKey = [Microsoft.Win32.RegistryKey]::OpenRemoteBaseKey(
         "CurrentUser", $computername)
 }
 else
 {
     Write-Error ("Please specify a fully-qualified registry path " +
         "(i.e.: HKLM:\Software) of the registry key to open.")
     return
 }
 
 $key = $baseKey.OpenSubKey($matches[1])
 
 foreach($subkeyName in $key.GetSubKeyNames())
 {
     $subkey = $key.OpenSubKey($subkeyName)
 
     $returnObject = [PsObject] $subKey
     $returnObject | Add-Member NoteProperty PsChildName $subkeyName
     $returnObject | Add-Member NoteProperty Property $subkey.GetValueNames()
 
     $returnObject
 
     $subkey.Close()
 }
 
 $key.Close()
 $baseKey.Close()
`

