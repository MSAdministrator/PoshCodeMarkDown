---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 1193
Published Date: 
Archived Date: 2010-07-17t01
---

# new-scriptcmdlet - 

## Description

a script to generate advanced functions that wrap cmdlets so you can tweak them, add features, etc. from the powershell team blog

## Comments



## Usage



## TODO



## script

`new-scriptcmdlet`

## Code

`#
 #
 ########################################################################################
 [CmdletBinding(DefaultParameterSetName="Type")]
 PARAM(
    [Parameter(ParameterSetName="Type",ValueFromPipeline=$true,Position=1)]
    [Type]
    $type
 ,
    [Parameter(ParameterSetName="CommandInfo",ValueFromPipeline=$true,Position=1)]
    [Management.Automation.CmdletInfo]
    $commandInfo
 ,
    [Parameter(Position=0)]
    [string]
    $name
 )
 
     Process
     {
         if (! $type) {
             if ($commandInfo.ImplementingType) { $type = $commandInfo.ImplementingType }
         }
 
         if ((! $type) -and (! $commandInfo)) {
 @"
 $(if ($name) { 'function ' + $name + '() {' })
 [CmdletBinding()]
 param   ()
 begin   {}
 process {}
 end     {}
 $(if ($name) {'}' })
 "@
         } else {
             if (! ($type.IsSubclassOf([Management.Automation.Cmdlet]))) {
                 throw "Must provide a cmdlet to create a proxy"
             }
 
             $commandMetaData = New-Object Management.Automation.CommandMetadata $type
             $proxyCommand =
 @"
 $(if ($name) { 'function ' + $name + '() {' })
 $([Management.Automation.ProxyCommand]::Create($commandMetaData))
 $(if ($name) {'}' })
 "@
 
             $executionContext.InvokeCommand.NewScriptBlock($proxyCommand)
         }
     }
 #}
`

