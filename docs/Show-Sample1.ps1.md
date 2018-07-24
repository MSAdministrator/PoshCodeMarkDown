---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2669
Published Date: 2011-05-10t20
Archived Date: 2011-05-16t21
---

# show-sample1 - 

## Description

a demonstration of how to do menus and commands in showui (it works against the latest changeset �d7ad095858eb� right now, so you can just hit the download link on the right of that page).

## Comments



## Usage



## TODO



## function

``

## Code

`#
 #
 if(!(Get-Command New-System.Windows.Input.CommandBinding -ErrorAction SilentlyContinue)) {
    Add-UIFunction -Type System.Windows.Input.CommandBinding
 }
 
 Show -Width 300 -Height 150 {
    DockPanel {
       Menu -DockPanel-Dock Top -Height 20 {
          MenuItem -Header "_File" {
             MenuItem -Header "_New" -Command ([system.windows.input.applicationcommands]::new)
             MenuItem -Header "_Open"
             MenuItem -Header "_Save"
             MenuItem -Header "Save _As"
             Separator
             MenuItem -Header "_Print"
             Separator
             MenuItem -Header "E_xit"
          }
          MenuItem -Header "_Edit" {
             MenuItem -Header "_Undo"
             Separator
             MenuItem -Header "Cu_t"
             MenuItem -Header "_Copy"
             MenuItem -Header "_Paste"
             MenuItem -Header "De_lete"
             Separator
             MenuItem -Header "_Find"
             MenuItem -Header "Find _Next"
             MenuItem -Header "_Replace"
             MenuItem -Header "_Go To"
             Separator
             MenuItem -Header "Select _All"
             MenuItem -Header "Time/_Date"
             
          }
          MenuItem -Header "F_ormat" {
             MenuItem -Header "_Word Wrap"
             MenuItem -Header "_Font"
          }
          MenuItem -Header "_View" {
             MenuItem -Header "_Status Bar"
          }
          MenuItem -Header "_Help" {
             MenuItem -Header "_About"
          }
       }
       
       TextBox -Name "Content"
    }
    
    
    $this.CommandBindings.Add( (
       CommandBinding -Command ([system.windows.input.applicationcommands]::new) `
                   -On_CanExecute { param($sender, $e) $e.CanExecute = $true }   `
                   -On_Executed { (Select-UIElement $this Content).Text = ""; } 
    ) ) | Out-Null
 }
`

