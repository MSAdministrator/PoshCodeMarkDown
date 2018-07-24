---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 4132
Published Date: 2014-04-25t17
Archived Date: 2017-01-08t16
---

# start-demo 1 for ps3 ise - 

## Description

the first version of a start-demo script (module) for ise from powershell 3+

## Comments



## Usage



## TODO



## function

`start-demo`

## Code

`#
 #
 function Start-Demo {
     param(
        $Text = $PSISE.CurrentFile.Editor.Text
     )
     $psISE.CurrentPowerShellTab.ConsolePane.Clear()
 
     $Tokens =  $Errors = $null
     $Script:Editor = $PSISE.CurrentFile.Editor
     $Script:Text = $Script:Editor.Text
     $AST = [System.Management.Automation.Language.Parser]::ParseInput( $Text, [ref]$Tokens, [ref]$Errors )
     if($Errors) { $Errors | Write-Error }
     $Script:Statements = $AST.EndBlock.Statements.Extent
     $Script:Index = $Script:Offset = 0
 
     $Function:Prompt = { PreDemoPrompt; Pop-Demo }
 }
 
 function Stop-Demo {
     $Function:Prompt = $function:PreDemoPrompt
 }
 
 function Pop-Demo {
     if($Script:Index -lt $Script:Statements.Count) {
         $Statement = $Script:Statements[$Script:Index]
         
         $DemoStep = $Statement.Text
 
         $Script:Editor.Select($Statement.StartLineNumber, $Statement.StartColumnNumber, $Statement.EndLineNumber, $Statement.EndColumnNumber)
         $Script:Offset = $Statement.EndOffset + 1
         $psISE.CurrentPowerShellTab.ConsolePane.InputText = $DemoStep
         $psISE.CurrentPowerShellTab.ConsolePane.Focus()
         $Script:Index += 1
     } else {
         Stop-Demo
     }
 }
 
 $function:PreDemoPrompt = $function:Prompt
 
 if(!($psISE.CurrentPowerShellTab.AddOnsMenu.Submenus.DisplayName -eq "Start Demo")) {
     try {
         $psISE.CurrentPowerShellTab.AddOnsMenu.Submenus.Add('Start Demo',{Start-Demo},"F6")
     } catch [System.Management.Automation.MethodInvocationException] {
         $psISE.CurrentPowerShellTab.AddOnsMenu.Submenus.Add('Start Demo',{Start-Demo},$null)
     }
 }
`

