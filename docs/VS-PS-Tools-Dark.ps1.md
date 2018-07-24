---
Author: oisin grehan
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 4609
Published Date: 2015-11-15t03
Archived Date: 2015-07-24t21
---

# vs ps tools dark - 

## Description

if you use the dark color scheme in vs, you’ll find that the powershell tools syntax highlighting is unusable. while it is possible to update the colors by hand in visual studio’s options, this is a pain in the ass. here’s a script to run in the nuget console that will set the token colors to something usable. the scheme is persistent.

## Comments



## Usage



## TODO



## function

`get-color`

## Code

`#
 #
 
 <#
 function get-color {
     $cdlg = new-object system.windows.forms.colordialog
     if ($cdlg.showdialog() -eq "OK") {
         $cdlg.color
     }
     $cdlg.dispose()
 }
 #>
 
 $map = @{}
 
 $tokens = $dte.Properties("FontsAndColors", "TextEditor").Item("FontsAndColorsItems").object
 $tokens | ? name -like "PowerShell *" | % { $_.Foreground = [system.drawing.colortranslator]::ToOle($map[$_.name]) }
`

