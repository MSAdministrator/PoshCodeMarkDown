---
Author: dmitry sotnikov
Publisher: 
Copyright: 
Email: 
Version: 1.2
Encoding: ascii
License: cc0
PoshCode ID: 984
Published Date: 2010-03-30t08
Archived Date: 2016-03-05t14
---

# set-powerguiwelcomepage - 

## Description

this script customizes the welcome screen which powergui admin console displays on start-up or when a folder is selected in the left-hand tree.

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 ########################################################
 ########################################################
 ########################################################
 #
 ########################################################
 
 param ([string] $mhtpath, [bool] $ResetToDefault = $false)
 
 if (( get-process Quest.PowerGUI -ErrorAction SilentlyContinue ) -ne $null) { 
 	throw "Please close the PowerGUI administrative console before running this script" 
 }
 
 $cfgpath = "$($env:APPDATA)\Quest Software\PowerGUI\quest.powergui.xml"
 
 Copy-Item $cfgpath "$cfgpath.backupconfig"
 
 $xml = [xml]$(Get-Content $cfgpath)
 
 
 if ( $ResetToDefault ) {
 
 	$node = $xml.SelectSingleNode("//container[@id='4b510268-a4eb-42e0-9276-06223660291d']")
 	if ($node -eq $null) {
 		"Configuration is already using default homepage. No rollback required."
 	} else {
 		$xml.SelectSingleNode("/configuration/items").RemoveChild($node) | Out-Null 
 		$xml.Save($cfgpath)
 		"SUCCESS: Successfully rolled PowerGUI back to default welcome page."
 	}
 		
 } else {
 
 	
 	if ( $mhtpath -eq $null ) {
 		$mhtpath = Read-Host "Please provide path to the MHT file"
 	}
 	$mhtfile = Get-ChildItem $mhtpath
 	if ( $mhtfile -eq $null) { 
 		throw "MHT file $mhtpath not found. Please verify the script parameter." 
 	}
 	if ( $mhtfile.Extension -ne ".mht" ) {
 		throw "File $mhtpath is not an MHT file. Only MHT files are supported." 
 	}
 	
 	
 	$node = $xml.SelectSingleNode("//container[@id='4b510268-a4eb-42e0-9276-06223660291d']")
 	if ($node -eq $null) {
 		$node = $xml.CreateElement("container")
 		
 		$node.SetAttribute("id", "4b510268-a4eb-42e0-9276-06223660291d")
 		$node.SetAttribute("name", "Home Page")
 		
 		$node.AppendChild($xml.CreateElement("value")) | Out-Null 
 		$xml.SelectSingleNode("/configuration/items").AppendChild($node) | Out-Null 
 	}
 	
 	$node.Value = [String] $mhtpath
 	$xml.Save($cfgpath)
 	"SUCCESS: $mhtpath is now set as your custom welcome screen."
 }
`

