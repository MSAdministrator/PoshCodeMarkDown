---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 1.5
Encoding: ascii
License: cc0
PoshCode ID: 4421
Published Date: 2014-08-28t06
Archived Date: 2017-05-22t03
---

# show-sample2 - 

## Description

a demonstration of how to do menus and commands in showui that works with showui 1.4

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 
 Window -Width 300 -Height 150 {
    DockPanel {
       Menu -Dock Top -Height 20 {
          MenuItem -Header "_File" {
             MenuItem -Command "New"
             MenuItem -Command "Open"
             MenuItem -Command "Save"
             MenuItem -Command "SaveAs"
             Separator
             MenuItem -Command "Print"
             Separator
             MenuItem -Header "E_xit" -On_Click { Close-Control $window }
          }
          MenuItem -Header "_Edit" {
             MenuItem -Command "Undo"
             MenuItem -Command "Redo"
             Separator
             MenuItem -Command "Select All"
             MenuItem -Command "Cut"
             MenuItem -Command "Copy"
             MenuItem -Command "Paste"
             MenuItem -Command "Delete"
             Separator
             MenuItem -Command "Find"
             MenuItem -Command "Replace"
          }
          MenuItem -Header "F_ormat" {
             MenuItem -Header "_Bold" -Command "ToggleBold"
             MenuItem -Header "_Italic" -Command "ToggleItalic"
          }
          MenuItem -Header "_View" {
             MenuItem -Header "_Status Bar"
             MenuItem -Header "_Word Wrap"
          }
          MenuItem -Header "_Help" {
             MenuItem -Header "_About"
          }
       }
       
       RichTextBox | % { $_.Name = "Content"; $_ }
 
    } -On_Load {
       $this.CommandBindings.Add( (
          CommandBinding -Command "New" `
                      -On_CanExecute {$_.CanExecute = $Content.Document.Blocks.Count -gt 0 }   `
                      -On_Executed { $Content.Document.Blocks.Clear() } 
       ) ) | Out-Null
    }
 } -Show
 
`

