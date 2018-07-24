---
Author: lee holmes
Publisher: 
Copyright: 
Email: 
Version: 3.5
Encoding: ascii
License: cc0
PoshCode ID: 4966
Published Date: 2016-03-08t09
Archived Date: 2016-03-18t21
---

# get-installedsoftware.ps - 

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
 
 Lists installed software on the current computer.
 
 .EXAMPLE
 
 Get-InstalledSoftware *Frame* | Select DisplayName
 
 DisplayName
 -----------
 Microsoft .NET Framework 3.5 SP1
 Microsoft .NET Framework 3.5 SP1
 Hotfix for Microsoft .NET Framework 3.5 SP1 (KB953595)
 Hotfix for Microsoft .NET Framework 3.5 SP1 (KB958484)
 Update for Microsoft .NET Framework 3.5 SP1 (KB963707)
 
 #>
 
 param(
     $DisplayName = "*"
 )
 
 Set-StrictMode -Off
 
 $keys =
     Get-ChildItem HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall
 
 $items = $keys | Foreach-Object { Get-ItemProperty $_.PsPath }
 
 foreach($item in $items)
 {
     if(($item.DisplayName) -and ($item.DisplayName -like $displayName))
     {
         $item
     }
 }
`

