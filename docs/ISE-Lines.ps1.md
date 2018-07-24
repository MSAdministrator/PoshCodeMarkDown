---
Author: poetter
Publisher: 
Copyright: 
Email: 
Version: 1.1
Encoding: ascii
License: cc0
PoshCode ID: 784
Published Date: 2009-01-04t09
Archived Date: 2016-07-10t07
---

# ise-lines - 

## Description

ise-lines module v 1.1 ( conflate-line improved )

## Comments



## Usage



## TODO



## module

`duplicate-line`

## Code

`#
 #
 ##############################################################################################################
 ##############################################################################################################
 
 ##############################################################################################################
 ##############################################################################################################
 function Duplicate-Line
 {
     $editor = $psISE.CurrentOpenedFile.Editor
     $caretLine = $editor.CaretLine
     $caretColumn = $editor.CaretColumn
     $text = $editor.Text.Split("`n")
     $line = $text[$caretLine -1]
     $newText = $text[0..($caretLine -1)]
     $newText += $line
     $newText += $text[$caretLine..($text.Count -1)]
     $editor.Text = [String]::Join("`n", $newText)
     $editor.SetCaretPosition($caretLine, $caretColumn)
 }
 
 ##############################################################################################################
 ##############################################################################################################
 function Conflate-Line
 {
     $editor = $psISE.CurrentOpenedFile.Editor
     $caretLine = $editor.CaretLine
     $caretColumn = $editor.CaretColumn
     $text = $editor.Text.Split("`n")
     if ( $caretLine -ne $text.Count )
     {
         $line = $text[$caretLine -1] + $text[$caretLine] -replace ("(``)?`r", "")
         $newText = @()
         if ( $caretLine -gt 1 )
         {
             $newText = $text[0..($caretLine -2)]
         }
         $newText += $line
         if ( $caretLine -ne $text.Count - 1)
         {
             $newText += $text[($caretLine +1)..($text.Count -1)]
         }
         $editor.Text = [String]::Join("`n", $newText)
         $editor.SetCaretPosition($caretLine, $caretColumn)
     }
 }
 
 ##############################################################################################################
 ##############################################################################################################
 function MoveUp-Line
 {
     $editor = $psISE.CurrentOpenedFile.Editor
     $caretLine = $editor.CaretLine
     if ( $caretLine -ne 1 )
     {
         $caretColumn = $editor.CaretColumn
         $text = $editor.Text.Split("`n")
         $line = $text[$caretLine -1]
         $lineBefore = $text[$caretLine -2]
         $newText = @()
         if ( $caretLine -gt 2 )
         {
             $newText = $text[0..($caretLine -3)]
         }
         $newText += $line
         $newText += $lineBefore
         if ( $caretLine -ne $text.Count )
         {
             $newText += $text[$caretLine..($text.Count -1)]
         }
         $editor.Text = [String]::Join("`n", $newText)
         $editor.SetCaretPosition($caretLine - 1, $caretColumn)
     }
 }
 
 ##############################################################################################################
 ##############################################################################################################
 function MoveDown-Line
 {
     $editor = $psISE.CurrentOpenedFile.Editor
     $caretLine = $editor.CaretLine
     $caretColumn = $editor.CaretColumn
     $text = $editor.Text.Split("`n")
     if ( $caretLine -ne $text.Count )
     {
         $line = $text[$caretLine -1]
         $lineAfter = $text[$caretLine]
         $newText = @()
         if ( $caretLine -ne 1 )
         {
             $newText = $text[0..($caretLine -2)]
         }
         $newText += $lineAfter
         $newText += $line
         if ( $caretLine -lt $text.Count -1 )
         {
             $newText += $text[($caretLine +1)..($text.Count -1)]
         }
         $editor.Text = [String]::Join("`n", $newText)
         $editor.SetCaretPosition($caretLine +1, $caretColumn)
     }
 }
 
 ##############################################################################################################
 ##############################################################################################################
 function Delete-TrailingBlanks
 {
     $editor = $psISE.CurrentOpenedFile.Editor
     $caretLine = $editor.CaretLine
     $newText = @()
     foreach ( $line in $editor.Text.Split("`n") )
     {
         $newText += $line -replace ("\s+$", "")
     }
     $editor.Text = [String]::Join("`n", $newText)
     $editor.SetCaretPosition($caretLine, 1)
 }
 
 ##############################################################################################################
 ##############################################################################################################
 if (-not( $psISE.CustomMenu.Submenus | where { $_.DisplayName -eq "Line" } ) )
 {
     $lineMenu = $psISE.CustomMenu.Submenus.Add("_Lines", $null, $null)
     $null = $lineMenu.Submenus.Add("Duplicate Line", {Duplicate-Line}, "Ctrl+Alt+D")
     $null = $lineMenu.Submenus.Add("Conflate Line", {Conflate-Line}, "Ctrl+Alt+J")
     $null = $lineMenu.Submenus.Add("Move Up Line", {MoveUp-Line}, "Ctrl+Shift+Up")
     $null = $lineMenu.Submenus.Add("Move Down Line", {MoveDown-Line}, "Ctrl+Shift+Down")
     $null = $lineMenu.Submenus.Add("Delete Trailing Blanks", {Delete-TrailingBlanks}, "Ctrl+Shift+Del")
 }
`

