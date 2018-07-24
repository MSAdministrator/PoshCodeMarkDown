---
Author: lee holmes
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6090
Published Date: 2016-11-16t11
Archived Date: 2016-05-24t22
---

# read-inputbox.ps1 - 

## Description

from windows powershell cookbook (o’reilly) by lee holmes

## Comments



## Usage



## TODO



## 

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
 
 Read user input from a visual InputBox
 
 .EXAMPLE
 
 PS >$caption = "Please enter your name"
 PS >$name = Read-InputBox $caption
 
 #>
 
 param(
     [string] $Title = "Input Dialog"
 )
 
 Set-StrictMode -Version Latest
 
 Add-Type -Assembly System.Windows.Forms
 
 $form = New-Object Windows.Forms.Form
 $form.Size = New-Object Drawing.Size @(400,100)
 $form.FormBorderStyle = "FixedToolWindow"
 
 $textbox = New-Object Windows.Forms.TextBox
 $textbox.Top = 5
 $textbox.Left = 5
 $textBox.Width = 380
 $textbox.Anchor = "Left","Right"
 $form.Text = $Title
 
 $buttonPanel = New-Object Windows.Forms.Panel
 $buttonPanel.Size = New-Object Drawing.Size @(400,40)
 $buttonPanel.Dock = "Bottom"
 
 $cancelButton = New-Object Windows.Forms.Button
 $cancelButton.Text = "Cancel"
 $cancelButton.DialogResult = "Cancel"
 $cancelButton.Top = $buttonPanel.Height - $cancelButton.Height - 10
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
 
 $form.Controls.Add($buttonPanel)
 $form.Controls.Add($textbox)
 $form.AcceptButton = $okButton
 $form.CancelButton = $cancelButton
 $form.Add_Shown( { $form.Activate(); $textbox.Focus() } )
 
 $result = $form.ShowDialog()
 
 if($result -eq "OK")
 {
     $textbox.Text
 }
`

