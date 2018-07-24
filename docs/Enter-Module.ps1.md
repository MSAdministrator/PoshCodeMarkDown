---
Author: lee holmes
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2142
Published Date: 2011-09-09t21
Archived Date: 2016-05-17t13
---

# enter-module.ps1 - 

## Description

from windows powershell cookbook (o’reilly) by lee holmes

## Comments



## Usage



## TODO



## module

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
 
 Lets you examine internal module state and functions by executing user
 input in the scope of the supplied module.
 
 .EXAMPLE
 
 PS >Import-Module PersistentState
 PS >Get-Module PersistentState
 
 ModuleType Name                      ExportedCommands
 ---------- ----                      ----------------
 Script     PersistentState           {Set-Memory, Get-Memory}
 
 
 PS >"Hello World" | Set-Memory
 PS >$m = Get-Module PersistentState
 PS >Enter-Module $m
 PersistentState: dir variable:\mem*
 
 Name                           Value
 ----                           -----
 memory                         {Hello World}
 
 PersistentState: exit
 PS >
 
 #>
 
 param(
     [System.Management.Automation.PSModuleInfo] $Module
 )
 
 Set-StrictMode -Version Latest
 
 $userInput = Read-Host $($module.Name)
 while($userInput -ne "exit")
 {
     $scriptblock = [ScriptBlock]::Create($userInput)
     & $module $scriptblock
 
     $userInput = Read-Host $($module.Name)
 }
`

