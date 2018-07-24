---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2666
Published Date: 2011-05-09t08
Archived Date: 2015-12-29t02
---

# get-uiinput - 

## Description

this is a flexible multiple-input prompt for show-ui (it works against the latest changeset ‘d7ad095858eb’ right now, so you can just hit the download link on the right)

## Comments



## Usage



## TODO



## function

`get-uiinput`

## Code

`#
 #
 function Get-UIInput {
       #.Synopsis
       #.Parameter PromptText
       #.Example
       Param([string[]]$PromptText = "Please enter your name:")
 
       Show-UI -Parameters @(,$PromptText) {
          Param([string[]]$PromptText)
 
          $global:UIInputWindow = $this
          function global:Get-UIInputOutput {
             $stack = Select-UIElement $UIInputWindow "Wrapper"
             $Output = @{}
             $inputs = $stack.Children
             while($inputs.Count) {
                $label, $value, $inputs = $inputs
                $Output.($label.Content) = $value.Text
             }
             Write-UIOutput $Output
          }
          
             Grid -Margin 10 -Name "Wrapper" -Columns Auto,150 -Rows (@("Auto") * ($PromptText.Count + 1)) {
 
                for($i=0;$i -lt $PromptText.Count;$i++) {
                   Label   -Grid-Row $i $PromptText[$i]
                   TextBox -Grid-Row $i -Grid-Column 1 -Width 150 -On_KeyDown { 
                      if($_.Key -eq "Return") { 
                         Get-UIInputOutput
                         $UIInputWindow.Close()
                      }
                   }
                }
                Button "Ok" -Grid-Row $PromptText.Count -Grid-Column 1 -Width 60 -On_Click { 
                   Get-UIInputOutput
                   $UIInputWindow.Close()
                }
             }
          }
       } -On_Load { (Select-UIElement $this "Wrapper").Children[1].Focus() }`
       -WindowStyle None -AllowsTransparency `
       -On_PreviewMouseLeftButtonDown { 
          if($_.Source -notmatch ".*\.(TextBox|Button)") 
          {
             $ShowUI.ActiveWindow.DragMove() 
          }
       }
    }
`

