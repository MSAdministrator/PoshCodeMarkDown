---
Author: doug finke
Publisher: 
Copyright: 
Email: 
Version: 1.0
Encoding: ascii
License: cc0
PoshCode ID: 1728
Published Date: 
Archived Date: 2010-07-17t01
---

# ise-filename - 

## Description

ise-filename module v 1.0

## Comments



## Usage



## TODO



## module

`get-isefullpath`

## Code

`#
 #
 ##############################################################################################################
 ##
 ##
 ##
 ##############################################################################################################
 ##############################################################################################################
 
 ##############################################################################################################
 ##############################################################################################################
 Function Get-ISEFullPath {
     $psISE.CurrentFile.FullPath
 }
 
 ##############################################################################################################
 ##############################################################################################################
 Function Copy-ISEFullPath {
     (Get-ISEFullPath) | Copy-ToClipBoard 
 }
 
 ##############################################################################################################
 ##############################################################################################################
 Function Copy-ISEPath {
    Split-Path (Get-ISEFullPath) | Copy-ToClipBoard 
 }
 
 ##############################################################################################################
 ##############################################################################################################
 Function Copy-ISEFileName {
    Split-Path -Leaf (Get-ISEFullPath) | Copy-ToClipBoard 
 }
 
 ##############################################################################################################
 ##############################################################################################################
 Filter Copy-ToClipBoard {
     $dataObject = New-Object Windows.DataObject 
     $dataObject.SetText($_, [Windows.TextDataFormat]"UnicodeText") 
     [Windows.Clipboard]::SetDataObject($dataObject, $true)
 }
 
 ##############################################################################################################
 ##############################################################################################################
 if (-not( $psISE.CurrentPowerShellTab.AddOnsMenu.Submenus | where { $_.DisplayName -eq "File/Path Names" } ) )
 {
 	$filenameMenu = $psISE.CurrentPowerShellTab.AddOnsMenu.SubMenus.Add("_File/Path Names",$null,$null) 
 	$null = $filenameMenu.Submenus.Add("Copy FullPath", {Copy-ISEFullPath}, "Ctrl+Alt+A")
 	$null = $filenameMenu.Submenus.Add("Copy Path", {Copy-ISEPath}, "Ctrl+Alt+P")
 	$null = $filenameMenu.Submenus.Add("Copy Filename", {Copy-ISEFileName}, "Ctrl+Alt+F")
 }
 
 #}
`

