---
Author: lee holmes
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5397
Published Date: 2016-09-03t21
Archived Date: 2016-05-25t11
---

# get-remoteregistrykeypro - 

## Description

from windows powershell cookbook (o’reilly) by lee holmes, updated for 32/64 bit view by burt harris

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
 ##
 ##############################################################################
 
 <#
 
 .SYNOPSIS
 
 Get the value of a remote registry key property
 
 .EXAMPLE
 
 PS >$registryPath =
      "HKLM:\software\Microsoft\PowerShell\1\ShellIds\Microsoft.PowerShell"
 PS >Get-RemoteRegistryKeyProperty LEE-DESK $registryPath ExecutionPolicy
 
 .EXAMPLE
 
     Get-RemoteRegistryKeyProperty Delta 'HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion' -View Registry64
 
     Get-RemoteRegistryKeyProperty Delta 'HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion' -View Registry32
 
 #>
 
 param(
     [Parameter(Mandatory = $true)]
     $ComputerName,
 
     [Parameter(Mandatory = $true)]
     $Path,
 
     $Property = "*",
 
     [Microsoft.Win32.RegistryView]$View = 'Default'
 )
 
 Set-StrictMode -Version Latest
 
 if($path -match "^HKLM:\\(.*)")
 {
     $baseKey = [Microsoft.Win32.RegistryKey]::OpenRemoteBaseKey(
         "LocalMachine", $computername, $view)
 }
 elseif($path -match "^HKCU:\\(.*)")
 {
     $baseKey = [Microsoft.Win32.RegistryKey]::OpenRemoteBaseKey(
         "CurrentUser", $computername, $view)
 }
 else
 {
     Write-Error ("Please specify a fully-qualified registry path " +
         "(i.e.: HKLM:\Software) of the registry key to open.")
     return
 }
 
 $key = $baseKey.OpenSubKey($matches[1])
 $returnObject = New-Object PsObject
 
 foreach($keyProperty in $key.GetValueNames())
 {
     if($keyProperty -like $property)
     {
         $returnObject |
             Add-Member NoteProperty $keyProperty $key.GetValue($keyProperty)
     }
 }
 
 $returnObject
 
 $key.Close()
 $baseKey.Close()
`

