---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 3835
Published Date: 2012-12-19t11
Archived Date: 2012-12-22t17
---

# add ? get-help -full - 

## Description

a crazy example of how you can extend powershell! this has two options for getting full help

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 $executionContext.SessionState.InvokeCommand.PreCommandLookupAction = {
     param($CommandName, $CommandLookupEventArgs)
 
     if($CommandName.StartsWith("?")) {
         $RealCommandName = $CommandName.TrimStart("?")
         $CommandLookupEventArgs.CommandScriptBlock = {
             Get-Help $RealCommandName -Full
         }.GetNewClosure()
     }
 }
 
 
 Write-Warning "DO NOT USE THIS POSTCOMMANDLOOKUPACTION EXCEPT FOR DEMONSTRATION"
 
 $executionContext.SessionState.InvokeCommand.PostCommandLookupAction = {
     param($CommandName, $CommandLookupEventArgs)
 
     if($CommandLookupEventArgs.CommandOrigin -eq "Runspace" -and $CommandName -ne "prompt" ) {
         $CommandLookupEventArgs.CommandScriptBlock = {
             if($Args.Length -eq 1 -and $Args[0] -eq "-??") {
                 Get-Help $CommandName -Full
             } else {
                 & $CommandName @args
             }
         }.GetNewClosure()
     }
 }
`

