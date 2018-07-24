---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 1824
Published Date: 
Archived Date: 2010-05-11t02
---

# powershell script config - 

## Description

stores script configuration on computer. this one is generic and would work with any powershell script/tool. a more powergui specific one can be found at

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 #################################################
 #
 #################################################
 
 $FolderName = "MyPowerGUIAddOn"
 $ConfigName = "MyAddOn.Config.xml"
 
 $parameters = @{}
 
 
 if ( Test-Path -Path "$($env:AppData)\$FolderName\$ConfigName") {
 	$parameters = Import-Clixml -Path "$($env:AppData)\$FolderName\$ConfigName"
 	  
   $parameters['A']
   $parameters['B']
 } else {
   $parameters['A'] = 5
   $parameters['B'] = "Qwerty"
 }
 
 
 if ( -not (Test-Path -Path "$($env:AppData)\$FolderName")) {
   mkdir "$($env:AppData)\$FolderName"
 }
 $parameters | Export-Clixml -Path "$($env:AppData)\$FolderName\$ConfigName"
`

