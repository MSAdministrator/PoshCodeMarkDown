---
Author: bernd kriszio
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 1297
Published Date: 2010-08-28t07
Archived Date: 2016-04-19t06
---

# start-selected - 

## Description

usuallay i use this to start my browser with a url in some powershell script or retrieved by some powershell script. this is an ise-extension.

## Comments



## Usage



## TODO



## function

`start-selected`

## Code

`#
 #
 if ($PSVersionTable.BuildVersion.build -le 7000)
 {
     function Start-Selected
     {
         $ed = $psise.CurrentOpenedFile.Editor
         start $ed.SelectedText
     }
 }
 else
 {
     function Start-Selected
     {
         $file = $psIse.CurrentPowerShellTab.Output.SelectedText
         if ($file -eq '')
         {
             $file = $psise.CurrentFile.Editor.SelectedText
         }
         if ($file)
         {
             start $file
         }    
     }
 }
`

