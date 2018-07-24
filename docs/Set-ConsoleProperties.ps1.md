---
Author: lee holmes
Publisher: 
Copyright: 
Email: 
Version: 1.0
Encoding: ascii
License: cc0
PoshCode ID: 6851
Published Date: 2017-04-20t19
Archived Date: 2017-04-24t03
---

# set-consoleproperties.ps - 

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
 
 Set-StrictMode -Version Latest
 
 Push-Location
 Set-Location HKCU:\Console
 New-Item '.\%SystemRoot%_system32_WindowsPowerShell_v1.0_powershell.exe'
 Set-Location '.\%SystemRoot%_system32_WindowsPowerShell_v1.0_powershell.exe'
 
 New-ItemProperty . ColorTable00 -type DWORD -value 0x00562401
 New-ItemProperty . ColorTable07 -type DWORD -value 0x00f0edee
 New-ItemProperty . FaceName -type STRING -value "Lucida Console"
 New-ItemProperty . FontFamily -type DWORD -value 0x00000036
 New-ItemProperty . FontSize -type DWORD -value 0x000c0000
 New-ItemProperty . FontWeight -type DWORD -value 0x00000190
 New-ItemProperty . HistoryNoDup -type DWORD -value 0x00000000
 New-ItemProperty . QuickEdit -type DWORD -value 0x00000001
 New-ItemProperty . ScreenBufferSize -type DWORD -value 0x0bb80078
 New-ItemProperty . WindowSize -type DWORD -value 0x00320078
 Pop-Location
`

