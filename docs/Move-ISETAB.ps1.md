---
Author: bernd kriszio
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 3455
Published Date: 2012-06-16t03
Archived Date: 2012-06-22t05
---

# move-isetab - 

## Description

a function to be called by ise addonmenu to move files to other tabs (runspaces)

## Comments



## Usage



## TODO



## function

`move-isetab`

## Code

`#
 #
 function Move-ISETab
 {
     param(
         $offset = 1
     )
 
     $curFile = $psISE.CurrentFile
     
     if ($curFile.IsSaved)
     {
         $curTabFiles = $psISE.CurrentPowerShellTab.Files
 
         $tabs = $psISE.PowerShellTabs
 
         $Fileindex = $null
         foreach ($i in 0..($curTabFiles.count -1))
         {
             if ($curTabFiles[$i].displayname -eq $curFile.DisplayName)
             {
                 $Fileindex = $i
                 break
             }
         }
         $file = $curFile.FullPath
 
         $Tabindex = $null
         foreach ($i in 0..($tabs.count -1))
         {
             if ($tabs[$i].displayname -eq $psISE.CurrentPowerShellTab.DisplayName)
             {
                 $Tabindex = $i
                 break
             }
         }
 
         $newTabIndex = ($TabIndex + $offset) %  $tabs.count
 
         $curTabFiles.removeAt($Fileindex)
         $newtab = $tabs[$newTabIndex]
         $psise.PowerShellTabs.SetSelectedPowerShellTab($newTab)
         $newObj = $tabs[$newTabIndex].files.add($file)
     }
     else
     {
         "You must save the file bevore moving it."
     }
  
 
 }
 
 $psISE.CurrentPowerShellTab.AddOnsMenu.Submenus.Add("Move-ToNextPowerShellTab",{Move-ISETab}, "Ctrl+Alt+Shift+RIGHT") | out-null
 $psISE.CurrentPowerShellTab.AddOnsMenu.Submenus.Add("Move-ToLastPowerShellTab",{Move-ISETab -1}, "Ctrl+Alt+Shift+LEFT") | out-null
`

