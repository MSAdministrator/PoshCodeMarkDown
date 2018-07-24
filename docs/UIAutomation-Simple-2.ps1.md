---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 1231
Published Date: 2009-07-24t14
Archived Date: 2012-12-28t07
---

# uiautomation simple 2 - 

## Description

more fun with s.w.a. @kryten68 – added transforms and examples

## Comments



## Usage



## TODO



## function

`select-window`

## Code

`#
 #
 [Reflection.Assembly]::Load("UIAutomationClient, Version=3.0.0.0, Culture=neutral, PublicKeyToken=31bf3856ad364e35")
 [Reflection.Assembly]::Load("UIAutomationTypes, Version=3.0.0.0, Culture=neutral, PublicKeyToken=31bf3856ad364e35")
 
 function Select-Window {
 PARAM( [string]$Text="*", [switch]$Recurse,
        [System.Windows.Automation.AutomationElement]$Parent = [System.Windows.Automation.AutomationElement]::RootElement ) 
 PROCESS {
    if($Recurse) {
       $Parent.FindAll( "Descendants", [System.Windows.Automation.Condition]::TrueCondition ) | 
       Where-Object { $_.GetCurrentPropertyValue([System.Windows.Automation.AutomationElementIdentifiers]::NameProperty)  -like $Text } |
       Add-Member -Name Window -Type ScriptProperty -Value {$this.GetCurrentPattern( [System.Windows.Automation.WindowPattern]::Pattern )} -Passthru |
       Add-Member -Name Transform -Type ScriptProperty -Value {$this.GetCurrentPattern( [System.Windows.Automation.TransformPattern]::Pattern )} -Passthru 
    } else {
       $Parent.FindAll( "Children", [System.Windows.Automation.Condition]::TrueCondition ) | 
       Where-Object { $_.GetCurrentPropertyValue([System.Windows.Automation.AutomationElementIdentifiers]::NameProperty)  -like $Text }|
       Add-Member -Name Window -Type ScriptProperty -Value {$this.GetCurrentPattern( [System.Windows.Automation.WindowPattern]::Pattern )} -Passthru |
       Add-Member -Name Transform -Type ScriptProperty -Value {$this.GetCurrentPattern( [System.Windows.Automation.TransformPattern]::Pattern )} -Passthru
    }
 }}
 
 
 
 
 #
`

