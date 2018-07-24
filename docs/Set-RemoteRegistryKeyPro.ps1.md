---
Author: lee holmes
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2222
Published Date: 2011-09-09t21
Archived Date: 2016-03-18t23
---

# set-remoteregistrykeypro - 

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
 
 Set the value of a remote registry key property
 
 .EXAMPLE
 
 PS >$registryPath =
     "HKLM:\software\Microsoft\PowerShell\1\ShellIds\Microsoft.PowerShell"
 PS >Set-RemoteRegistryKeyProperty LEE-DESK $registryPath `
       "ExecutionPolicy" "RemoteSigned"
 
 #>
 
 param(
     [Parameter(Mandatory = $true)]
     $ComputerName,
 
     [Parameter(Mandatory = $true)]
     $Path,
 
     [Parameter(Mandatory = $true)]
     $PropertyName,
 
     [Parameter(Mandatory = $true)]
     $PropertyValue
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
 
 $key = $baseKey.OpenSubKey($matches[1], $true)
 $key.SetValue($propertyName, $propertyValue)
 
 $key.Close()
 $baseKey.Close()
`

