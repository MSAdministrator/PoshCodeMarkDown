---
Author: chad miller
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2518
Published Date: 2011-02-24t07
Archived Date: 2016-07-02t06
---

# get-psexecutionpolicy - 

## Description

get-psexecutionpolicy.ps1 uses wmi to query remote registry for powershell path and exeuction policy settings. because wmi is used works from x86 to x64 machines.

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 
 param($computerName=$env:computerName)
 
 $reg = Get-WmiObject -List -Namespace root\default -ComputerName $ComputerName | Where-Object {$_.Name -eq "StdRegProv"}
 
 new-object psobject -property @{
 ComputerName=$ComputerName;
 Path=($reg.GetStringValue(2147483650,"SOFTWARE\Microsoft\PowerShell\1\ShellIds\Microsoft.PowerShell","Path")).sValue;
 Policy=($reg.GetStringValue(2147483650,"SOFTWARE\Microsoft\PowerShell\1\ShellIds\Microsoft.PowerShell","ExecutionPolicy")).sValue;
 x86Path=($reg.GetStringValue(2147483650,"SOFTWARE\Wow6432Node\Microsoft\PowerShell\1\ShellIds\Microsoft.PowerShell","Path")).sValue
 x86Policy=($reg.GetStringValue(2147483650,"SOFTWARE\Wow6432Node\Microsoft\PowerShell\1\ShellIds\Microsoft.PowerShell","ExecutionPolicy")).sValue
 }
`

