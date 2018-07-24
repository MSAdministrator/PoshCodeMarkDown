---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 1.0
Encoding: utf-8
License: cc0
PoshCode ID: 3440
Published Date: 
Archived Date: 2012-06-06t06
---

#  - 

## Description

sometimes our terminalserver sessions become unstable, session didn’t

## Comments



## Usage



## TODO



## module

``

## Code

`#
 #
 ---------------------logoff_clientside_interactive.ps1-----------------
 $ErrorActionPreference = "silentlycontinue"
 $mycreds = (Get-Credential)
 Invoke-Command  -ComputerName terminalserver -Credential $mycreds  {
 &"C:\Program Files\internal\logoff_serverside.ps1"
 }
 ---------------------logoff_clientside_interactive.ps1-----------------
 or even simpler
 ---------------------logoff_clientside_same_user.ps1-------------------
 $ErrorActionPreference = "silentlycontinue"
 Invoke-Command  -ComputerName terminalserver {
 &"C:\Program Files\internal\logoff_serverside.ps1"
 }
 ---------------------logoff_clientside_same_user.ps1-------------------
 
 
 ---------------------logoff_serverside.ps1-----------------------------
 
 Import-module PSTerminalServices
 $myId=[System.Security.Principal.WindowsIdentity]::GetCurrent()
 $data=$myID.name.split('\')
 $data=$data[1]
 Get-TSSession -ComputerName localhost -Filter {$_.Username -eq $data}  | Stop-TSSession �Force
 ---------------------logoff_serverside.ps1-----------------------------
`

