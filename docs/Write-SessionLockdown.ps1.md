---
Author: anonymous
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 4312
Published Date: 2013-07-19t09
Archived Date: 2013-07-25t10
---

# write-sessionlockdown.ps - 

## Description

this file was uploaded by a powergui script editor add-on.

## Comments



## Usage



## TODO



## function

`write-sessionlockdown`

## Code

`#
 #
 function Write-SessionLockdown
 {
     
 	<#
     .Synopsis
         Writes a script to lock down a session to a list of required commands
     .Description
         Writes a script to lock down a session to a list of required commands.  
         This script can be at the end of a constrained runspace definiton
     .Example
         Write-SessionLockdown -RequiredCommands 'Write-Host'
     #>
 	
     param(
     [Parameter(Mandatory=$true,Position=0,ValueFromPipelineByPropertyName=$true)]
     [string[]]
     $RequiredCommands
     )
     
     process {
 $ofs = "','"
 "`$requiredCommands = '$requiredCommands'
 " + @'
 $ExecutionContext.SessionState.Scripts.Clear()
 $ExecutionContext.SessionState.Applications.Clear()
 $commandsToProxyNames = @([Management.Automation.CommandMetaData]::GetRestrictedCommands("RemoteServer").Keys)
 $requiredCommands += $commandsToProxyNames 
     
 Get-Command |
     ForEach-Object {
         $cmd = $_
         if ($requiredCommands -notcontains $cmd.Name) {
             $Cmd.Visibility = 'Private'
         }
         if ($commandsToProxyNames -contains $cmd.Name) {
             $cmdMd = [Management.Automation.CommandMetaData]::GetRestrictedCommands("RemoteServer")[$cmd.Name]
             $proxy = [Management.Automation.ProxyCommand]::Create($cmdMd)
             . ([ScriptBLock]::Create(
 "function $($cmd.Name) {
     $proxy
 }
 "            
 ))
             $cmd.Visibility = 'Private'
         }        
     }    
 
 $ExecutionContext.SessionState.LanguageMode = 'NoLanguage'
 '@
     
     }
 }
`

