---
Author: nathan kasco
Publisher: 
Copyright: 
Email: 
Version: 1.0
Encoding: ascii
License: cc0
PoshCode ID: 6198
Published Date: 2016-01-31t02
Archived Date: 2016-03-19t08
---

# copy-itemgui - 

## Description

allows you to copy items and display the native windows copy dialogue instead of having to use -passthru or fuss with write-progress

## Comments



## Usage



## TODO



## function

`copy-itemgui`

## Code

`#
 #
 function Copy-ItemGUI
 {
 <#
 .Synopsis
    GUI display for copy-item function
 .DESCRIPTION
    Allows you to copy items and display the native windows copy dialogue instead of having to use -passthru or fuss with write-progress
 .EXAMPLE
    Copy-ItemGUI -Source $Folder -Destination "C:\SomeFolder"
 .NOTES
    v1.0 - 2016-01-30 - Nathan Kasco
 #>
     Param
     (
         $Source,
 
         [Parameter(Mandatory=$true, Position=1)]
         [string]
         $Destination
     )
 
     if(!(Test-Path $Destination)){
         break
     }
 
     $src = gci $Source
 
     $objShell = New-Object -ComObject "Shell.Application"
 
     $objFolder = $objShell.NameSpace($Destination) 
 
     $counter = ($src.Length) - 1
     foreach($file in $src){
         Write-Host -ForegroundColor Cyan "Copying file '"$file.name"' to ' $Destination '"
 
         try{
             $objFolder.CopyHere("$source\$file", "&H0&")
         }
         catch{
             Write-Error $_
         }
 
         Write-Host -ForegroundColor Green "Copy complete - Number of items remaining: $counter`n"
         $counter--
     }
 }
`

