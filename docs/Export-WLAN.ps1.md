---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 1.0
Encoding: ascii
License: cc0
PoshCode ID: 2474
Published Date: 
Archived Date: 2011-01-25t21
---

#  - 

## Description

powershell functions to export and import wlan profiles in windows vista/windows 7

## Comments



## Usage



## TODO



## function

`export-wlan`

## Code

`#
 #
 <#
 
  NAME: WLAN-functions.ps1
 
  AUTHOR: Jan Egil Ring
  EMAIL: jan.egil.ring@powershell.no
 
  COMMENT: PowerShell functions to export and import WLAN profiles in Windows Vista/Windows 7
 
           Required version: Windows PowerShell 2.0 (built-in to Windows 7)
 		  Usage: Either copy the functions directly into a PowerShell session, or dot-source the script
 		  to make the functions available (. .\WLAN-functions.ps1)
 		            
           For more information, see the following blog-post:
 		  http://blog.powershell.no/2011/01/23/export-and-import-wlan-profiles
       
  You have a royalty-free right to use, modify, reproduce, and
  distribute this script file in any way you find useful, provided that
  you agree that the creator, owner above has no warranty, obligations,
  or liability for such use.
 
  VERSION HISTORY:
  1.0 23.01.2011 - Initial release
  
 #>
 
 function Export-WLAN {
 <#
 .SYNOPSIS
 Exports all-user WLAN profiles
 .DESCRIPTION
 Exports all-user WLAN profiles to Xml-files to the specified directory using netsh.exe
 .PARAMETER XmlDirectory
 Directory to export Xml configuration-files to
 .EXAMPLE
 Export-WLAN -XmlDirectory c:\temp\wlan
 #>
 
 [CmdletBinding()]
     param (
         [parameter(Mandatory=$true)]
         [string]$XmlDirectory
 		)
 
 $wlans = netsh wlan show profiles | Select-String -Pattern "All User Profile" | Foreach-Object {$_.ToString()}
 $exportdata = $wlans | Foreach-Object {$_.Replace("    All User Profile     : ",$null)}
 $exportdata | ForEach-Object {netsh wlan export profile $_ $XmlDirectory}
 }
 
 function Import-WLAN {
 <#
 .SYNOPSIS
 Imports all-user WLAN profiles based on Xml-files in the specified directory
 .DESCRIPTION
 Imports all-user WLAN profiles based on Xml-files in the specified directory using netsh.exe
 .PARAMETER XmlDirectory
 Directory to import Xml configuration-files from
 .EXAMPLE
 Import-WLAN -XmlDirectory c:\temp\wlan
 #>
 
 [CmdletBinding()]
     param (
         [parameter(Mandatory=$true)]
         [string]$XmlDirectory
 		)
 
 Get-ChildItem $XmlDirectory | Where-Object {$_.extension -eq ".xml"} | ForEach-Object {netsh wlan add profile filename=($XmlDirectory+"\"+$_.name)}
 }
`

