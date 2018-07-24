---
Author: lee holmes
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2157
Published Date: 2011-09-09t21
Archived Date: 2016-03-18t23
---

# get-machinestartupshutdo - 

## Description

from windows powershell cookbook (o’reilly) by lee holmes

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 
 
 <#
 
 .SYNOPSIS
 
 Get the startup or shutdown scripts assigned to a machine
 
 .EXAMPLE
 
 Get-MachineStartupShutdownScript -ScriptType Startup
 Gets startup scripts for the machine
 
 #>
 
 param(
     [Parameter(Mandatory = $true)]
     [ValidateSet("Startup","Shutdown")]
     $ScriptType
 )
 
 Set-StrictMode -Version Latest
 
 $registryKey = "HKLM:\SOFTWARE\Policies\Microsoft\Windows\System\Scripts"
 
 if(-not (Test-Path $registryKey))
 {
     return
 }
 
 foreach($policy in Get-ChildItem $registryKey\$scriptType)
 {
     foreach($script in Get-ChildItem $policy.PsPath)
     {
         Get-ItemProperty $script.PsPath | Select Script,Parameters
     }
 }
`

