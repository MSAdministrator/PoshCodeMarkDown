---
Author: ravikanth chaganti
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2230
Published Date: 
Archived Date: 2010-09-22t05
---

# auto ise preferences - 

## Description

you can just copy paste this entire script to your powershell ise profile script. this script will take care of saving your ise preferences such as color schemes, font styles, etc. you can change any of these values and this script will take care of storing and re-storing those preferences everytime you open ise.

## Comments



## Usage



## TODO



## script

`new-eventaction`

## Code

`#
 #
 ##############################################################
 ##############################################################
 
 Add-Type -AssemblyName System.XML
 
 $xmlSerializer = New-Object System.Xml.Serialization.XmlSerializer($PSISE.Options.GetType())
 
 $file = $("$env:APPDATA\isePreferences.xml")
 
 function New-EventAction {
     param ([System.Management.Automation.PSEventArgs]$objEvent)
     $changedProperty = ($objEvent.SourceEventArgs.PropertyName).ToString()
     Write-Host "Value of $changedProperty changed to $($objEvent.SourceArgs.Get(0).$changedProperty)"
     $xmlWriter = [System.Xml.XmlTextWriter]::Create($file)
     $xmlSerializer.Serialize($xmlWriter,$psISE.Options)
     $xmlWriter.Close()
 }
 
 function Update-ISEOptions {
     If (!(Get-Item $file -ea SilentlyContinue)) {
             Write-Host "$file not found"
             return
     }
         $xmlReader = New-Object System.Xml.XmlTextReader($file)
         $newISEOptions = $xmlSerializer.Deserialize($xmlReader)        
         $psISE.Options.SelectedScriptPaneState=$newISEOptions.SelectedScriptPaneState
         $psISE.Options.ShowToolBar=$newISEOptions.ShowToolBar
         $psISE.Options.FontSize=$newISEOptions.FontSize
         $psISE.Options.FontName=$newISEOptions.FontName
         $psISE.Options.ErrorForegroundColor=$newISEOptions.ErrorForegroundColor
         $psISE.Options.ErrorBackgroundColor=$newISEOptions.ErrorBackgroundColor
         $psISE.Options.WarningForegroundColor=$newISEOptions.WarningForegroundColor
         $psISE.Options.WarningBackgroundColor=$newISEOptions.WarningBackgroundColor
         $psISE.Options.VerboseForegroundColor=$newISEOptions.VerboseForegroundColor
         $psISE.Options.VerboseBackgroundColor=$newISEOptions.VerboseBackgroundColor
         $psISE.Options.DebugForegroundColor=$newISEOptions.DebugForegroundColor
         $psISE.Options.DebugBackgroundColor=$newISEOptions.DebugBackgroundColor
         $psISE.Options.OutputPaneBackgroundColor=$newISEOptions.OutputPaneBackgroundColor
         $psISE.Options.OutputPaneTextBackgroundColor=$newISEOptions.OutputPaneTextBackgroundColor
         $psISE.Options.OutputPaneForegroundColor=$newISEOptions.OutputPaneForegroundColor
         $psISE.Options.CommandPaneBackgroundColor=$newISEOptions.CommandPaneBackgroundColor
         $psISE.Options.ScriptPaneBackgroundColor=$newISEOptions.ScriptPaneBackgroundColor
         $psISE.Options.ScriptPaneForegroundColor=$newISEOptions.ScriptPaneForegroundColor
         $psISE.Options.ShowWarningForDuplicateFiles=$newISEOptions.ShowWarningForDuplicateFiles
         $psISE.Options.ShowWarningBeforeSavingOnRun=$newISEOptions.ShowWarningBeforeSavingOnRun
         $psISE.Options.UseLocalHelp=$newISEOptions.UseLocalHelp
         $psISE.Options.CommandPaneUp = $newISEOptions.CommandPaneUp
         $xmlReader.Close()
     
 }
 
 if ($psise) {
     Update-ISEOptions
     Register-ObjectEvent -InputObject $psISE.Options -EventName PropertyChanged -Action { New-EventAction $Event } | Out-Null
 }
`

