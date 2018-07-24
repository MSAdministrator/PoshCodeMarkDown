---
Author: andy levy
Publisher: 
Copyright: 
Email: 
Version: 2.0
Encoding: ascii
License: cc0
PoshCode ID: 3997
Published Date: 2014-03-03t13
Archived Date: 2014-11-09t17
---

# short ps prompt - 

## Description

dynamically adjusts the length of the path displayed in your prompt based upon the width of the window

## Comments



## Usage



## TODO



## function

``

## Code

`#
 #
 <#
 .Synopsis
 	Dynamically shortens the prompt based upon window size
 .Notes
 	I got really annoyed by having my PowerShell prompt extend across 2/3 of my window when in a deeply-nested directory structure.
 	This shortens the prompt to roughly 1/3 of the window width, at a minimum showing the first and last piece of the path (usually the PSPROVIDER & the current directory)
 	Additional detail is added, starting at the current directory's parent and working up from there.
 	The omitted portion of the path is represented with an ellipsis (...)
 #>
 
 if(!$global:WindowTitlePrefix) {
          new-object Security.Principal.WindowsPrincipal (
             ).IsInRole([Security.Principal.WindowsBuiltInRole]::Administrator) )
    {
       $global:WindowTitlePrefix = "PoSh (ADMIN)"
    } else {
       $global:WindowTitlePrefix = "PoSh"
    }
 }
 
 function prompt {
     $host.ui.rawui.windowtitle = $global:WindowTitlePrefix + " - " + $(get-location);
 	
 	$winWidth = $host.UI.RawUI.WindowSize.Width;
     $maxPromptPath = [Math]::Round($winWidth/3);
 	
     if (-not ($winWidth -eq $null)) {
         $currPath = (get-location).path;
         if ($currPath.length -ge $maxPromptPath){
             $pathParts = $currPath.split([System.IO.Path]::DirectorySeparatorChar);
             $myPrompt = $pathParts[0] + [System.IO.Path]::DirectorySeparatorChar+ "..." + [System.IO.Path]::DirectorySeparatorChar + $pathParts[$pathParts.length - 1];
             $counter = $pathParts.length - 2;
             while( ($myPrompt.replace("...","..."+[System.IO.Path]::DirectorySeparatorChar+$pathParts[$counter]).length -lt $maxPromptPath) -and ($counter -ne 0)) {
                 $myPrompt = $myPrompt.replace("...","..."+[System.IO.Path]::DirectorySeparatorChar+$pathParts[$counter]);
                 $counter--;
             }      
             $($myPrompt) + ">";    
         } else{
             $(if (test-path variable:/PSDebugContext) { '[DBG]: ' } else { '' }) + 'PS ' + $(Get-Location) + $(if ($nestedpromptlevel -ge 1) { '>>' }) + '> '
         }
     }
 }
`

