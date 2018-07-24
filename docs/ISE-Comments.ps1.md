---
Author: poetter
Publisher: 
Copyright: 
Email: 
Version: 1.0
Encoding: ascii
License: cc0
PoshCode ID: 773
Published Date: 2009-01-02t18
Archived Date: 2017-05-17t23
---

# ise-comments - 

## Description

ise-comments module v 1.0

## Comments



## Usage



## TODO



## module

`convertto-blockcomment`

## Code

`#
 #
 ##############################################################################################################
 ##############################################################################################################
 
 ##############################################################################################################
 ##############################################################################################################
 function ConvertTo-BlockComment
 {
     $editor = $psISE.CurrentOpenedFile.Editor
     $CommentedText = "<#`n" + $editor.SelectedText + "#>"
     $editor.InsertText($CommentedText)
 }
 
 ##############################################################################################################
 ##############################################################################################################
 function ConvertTo-BlockUncomment
 {
     $editor = $psISE.CurrentOpenedFile.Editor
     $CommentedText = $editor.SelectedText -replace ("^<#`n", "")
     $CommentedText = $CommentedText -replace ("#>$", "")
     $editor.InsertText($CommentedText)
 }
 
 ##############################################################################################################
 ##############################################################################################################
 function ConvertTo-Comment
 {
     $editor = $psISE.CurrentOpenedFile.Editor
     $CommentedText = $editor.SelectedText.Split("`n")
     $editor.InsertText( "#" + ( [String]::Join("`n#", $CommentedText)))
 }
 
 ##############################################################################################################
 ##############################################################################################################
 function ConvertTo-Uncomment
 {
     $editor = $psISE.CurrentOpenedFile.Editor
     $CommentedText = $editor.SelectedText.Split("`n") -replace ( "^#", "" )
     $editor.InsertText( [String]::Join("`n", $CommentedText))
 }
 
 ##############################################################################################################
 ##############################################################################################################
 if (-not( $psISE.CustomMenu.Submenus | where { $_.DisplayName -eq "Comments" } ) )
 {
     $commentsMenu = $psISE.CustomMenu.Submenus.Add("_Comments", $null, $null)
     $null = $commentsMenu.Submenus.Add("Block Comment Selected", {ConvertTo-BlockComment}, "Ctrl+SHIFT+B")
     $null = $commentsMenu.Submenus.Add("Block Uncomment Selected", {ConvertTo-BlockUncomment}, "Ctrl+Alt+B")
     $null = $commentsMenu.Submenus.Add("Comment Selected", {ConvertTo-Comment}, "Ctrl+SHIFT+C")
     $null = $commentsMenu.Submenus.Add("Uncomment Selected", {ConvertTo-Uncomment}, "Ctrl+Alt+C")
 }
`

