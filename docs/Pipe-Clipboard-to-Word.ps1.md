---
Author: banker42
Publisher: 
Copyright: 
Email: 
Version: 12.0
Encoding: ascii
License: cc0
PoshCode ID: 6708
Published Date: 2017-01-20t20
Archived Date: 2017-05-11t14
---

# pipe clipboard to word - 

## Description

quick, dirty, and ugly way to output clipboard data to microsoft word.

## Comments



## Usage



## TODO



## function

``

## Code

`#
 #
 
 .Get-Hotfix | clip.exe | AutoWord
 
 .Get-Hotfix | Format-List | clip.exe | AutoWord
 
 #>
 
 
 Function Paste {
 
 Add-Type -AssemblyName System.Windows.Forms
 $P = New-Object System.Windows.Forms.TextBox
 $P.Multiline = $true
 $P.Paste()
 $P.Text
 
 }
 
 Function AutoWord {
 
 param ([string[]]$Paste)
 
 $Word = New-Object -ComObject Word.Application
 $Word.visible = $true
 
 if (($WordVersion -eq "12.0") -or ($WordVersion -eq "11.0"))
     {    
     $Word.DisplayAlerts = $false
     }
     else
     {
     $Word.DisplayAlerts = "wdAlertsNone"
     }
 
 $Paste = Paste
 $Document = $Word.Documents.Add()
 $Selection = $Word.Selection
 $Selection.TypeText("$Paste")
 
 
 }
`

