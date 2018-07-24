---
Author: lee holmes
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5195
Published Date: 2016-05-27t23
Archived Date: 2016-03-19t01
---

# select-graphicalfiltered - 

## Description

from windows powershell cookbook (o’reilly) by lee holmes

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 ##############################################################################
 ##
 ##
 ##
 ##############################################################################
 
 <#
 
 .SYNOPSIS
 
 Display a Windows Form to help the user select a list of items piped in.
 Any selected items get passed along the pipeline.
 
 .EXAMPLE
 
 dir | Select-GraphicalFilteredObject
 
   Directory: C:\
 
 Mode                LastWriteTime     Length Name
 ----                -------------     ------ ----
 d----         10/7/2006   4:30 PM            Documents and Settings
 d----         3/18/2007   7:56 PM            Windows
 
 #>
 
 Set-StrictMode -Version Latest
 
 $objectArray = @($input)
 
 if($objectArray.Count -eq 0)
 {
     Write-Error "This script requires pipeline input."
     return
 }
 
 Add-Type -Assembly System.Windows.Forms
 
 $form = New-Object Windows.Forms.Form
 $form.Size = New-Object Drawing.Size @(600,600)
 
 $listbox = New-Object Windows.Forms.CheckedListBox
 $listbox.CheckOnClick = $true
 $listbox.Dock = "Fill"
 $form.Text = "Select the list of objects you wish to pass down the pipeline"
 $listBox.Items.AddRange($objectArray)
 
 $buttonPanel = New-Object Windows.Forms.Panel
 $buttonPanel.Size = New-Object Drawing.Size @(600,30)
 $buttonPanel.Dock = "Bottom"
 
 $cancelButton = New-Object Windows.Forms.Button
 $cancelButton.Text = "Cancel"
 $cancelButton.DialogResult = "Cancel"
 $cancelButton.Top = $buttonPanel.Height - $cancelButton.Height - 5
 $cancelButton.Left = $buttonPanel.Width - $cancelButton.Width - 10
 $cancelButton.Anchor = "Right"
 
 $okButton = New-Object Windows.Forms.Button
 $okButton.Text = "Ok"
 $okButton.DialogResult = "Ok"
 $okButton.Top = $cancelButton.Top
 $okButton.Left = $cancelButton.Left - $okButton.Width - 5
 $okButton.Anchor = "Right"
 
 $buttonPanel.Controls.Add($okButton)
 $buttonPanel.Controls.Add($cancelButton)
 
 $form.Controls.Add($listBox)
 $form.Controls.Add($buttonPanel)
 $form.AcceptButton = $okButton
 $form.CancelButton = $cancelButton
 $form.Add_Shown( { $form.Activate() } )
 
 $result = $form.ShowDialog()
 
 if($result -eq "OK")
 {
     foreach($index in $listBox.CheckedIndices)
     {
         $objectArray[$index]
     }
 }
`

