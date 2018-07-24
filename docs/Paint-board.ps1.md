---
Author: greg zakharov
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 4030
Published Date: 2013-03-18t17
Archived Date: 2013-03-21t05
---

# paint board - 

## Description

just for fun (my original post http

## Comments



## Usage



## TODO



## function

``

## Code

`#
 #
 function frmMain_Show {
   Add-Type -AssemblyName PresentationFramework
 
   $win = New-Object Windows.Window
   $ink = New-Object Windows.Controls.InkCanvas
   #
   #
   $ink.MinWidth = $ink.MinHeight = 450
   #
   #
   $win.Content = $ink
   $win.SizeToContent = "WidthAndHeight"
   $win.Title = "Paint board"
   $win.WindowStartupLocation = "CenterScreen"
 
   [void]$win.ShowDialog()
 }
 
 frmMain_Show
`

