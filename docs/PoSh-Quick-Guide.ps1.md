---
Author: greg zakharov
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 4540
Published Date: 2014-10-22t09
Archived Date: 2014-01-25t04
---

# posh quick guide - 

## Description

fixed and redesigned

## Comments



## Usage



## TODO



## function

`add-aboutpages`

## Code

`#
 #
 function Add-AboutPages {
   $tsCbo_2.Items.Clear()
   $tsCbo_2.Items.AddRange(($arr | % {Split-Path -leaf $_}))
   $tsCbo_2.SelectedIndex = (Get-Random -max ($arr.Length))
 }
 
 function Add-CmdletPages {
   $tsCbo_2.Items.Clear()
   $tsCbo_2.Items.AddRange((Get-Command -com Cmdlet))
   $tsCbo_2.SelectedIndex = (Get-Random -max ($tsCbo_2.Items.Count - 1))
 }
 
 $tsCbo_1_SelectedIndexChanged= {
   switch ($tsCbo_1.SelectedIndex) {
     "0" {Add-CmdletPages; break}
     "1" {Add-AboutPages; break}
   }
 }
 
 $tsBRead_Click= {
   $txtRead.Clear()
   switch ($tsCbo_1.SelectedIndex) {
     "0" {
       foreach ($pco in (Get-Help $tsCbo_2.SelectedItem)) {
         $pco.Description | % {$txtRead.Text = $_.Text}
       }
     }
     "1" {
       $txtRead.Text = [IO.File]::ReadAllText($arr[$tsCbo_2.SelectedIndex], [Text.Encoding]::Default)
     }
   }
 }
 
 $mnuFont_Click= {
   (New-Object Windows.Forms.FontDialog) | % {
     $_.Font = "Lucida Console"
     $_.MaxSize = 12
     $_.ShowEffects = $false
     
     if ($_.ShowDialog() -eq [Windows.Forms.DialogResult]::OK) {
       $txtRead.Font = $_.Font
     }
   }
 }
 
 $mnuSPan_Click= {
   $toggle =! $mnuSPan.Checked
   $mnuSPan.Checked = $toggle
   $sbPanel.Visible = $toggle
 }
 
 function frmMain_Show {
   Add-Type -AssemblyName System.Windows.Forms
   [Windows.Forms.Application]::EnableVisualStyles()
   
   $ico = [Drawing.Icon]::ExtractAssociatedIcon(($PSHome + '\powershell.exe'))
   $arr = gci ($PSHome + '\' + (Get-Culture).TwoLetterISOLanguageName) -fi *.txt | % {$_.FullName}
   
   $frmMain = New-Object Windows.Forms.Form
   $mnuMain = New-Object Windows.Forms.MenuStrip
   $mnuFile = New-Object Windows.Forms.ToolStripMenuItem
   $mnuExit = New-Object Windows.Forms.ToolStripMenuItem
   $mnuView = New-Object Windows.Forms.ToolStripMenuItem
   $mnuFont = New-Object Windows.Forms.ToolStripMenuItem
   $mnuSPan = New-Object Windows.Forms.ToolStripMenuItem
   $tsTools = New-Object Windows.Forms.ToolStrip
   $tsLab_1 = New-Object Windows.Forms.ToolStripLabel
   $tsLab_2 = New-Object Windows.Forms.ToolStripLabel
   $tsCbo_1 = New-Object Windows.Forms.ToolStripComboBox
   $tsCbo_2 = New-Object Windows.Forms.ToolStripComboBox
   $tsSepar = New-Object Windows.Forms.ToolStripSeparator
   $tsBRead = New-Object Windows.Forms.ToolStripButton
   $txtRead = New-Object Windows.Forms.TextBox
   $sbPanel = New-Object Windows.Forms.StatusBar
   #
   #
   $mnuMain.Items.AddRange(@($mnuFile, $mnuView))
   #
   #
   $mnuFile.DropDownItems.AddRange(@($mnuExit))
   $mnuFile.Text = "&File"
   #
   #
   $mnuExit.ShortcutKeys = "Control", "X"
   $mnuExit.Text = "E&xit"
   $mnuExit.Add_Click({$frmMain.Close()})
   #
   #
   $mnuView.DropDownItems.AddRange(@($mnuFont, $mnuSPan))
   $mnuView.Text = "&View"
   #
   #
   $mnuFont.ShortcutKeys = "Control", "F"
   $mnuFont.Text = "&Font"
   $mnuFont.Add_Click($mnuFont_Click)
   #
   #
   $mnuSPan.Checked = $true
   $mnuSPan.ShortcutKeys = "Control", "B"
   $mnuSPan.Text = "Status &Bar Toggle"
   $mnuSPan.Add_Click($mnuSPan_Click)
   #
   #
   $tsTools.Items.AddRange(@($tsLab_1, $tsCbo_1, $tsSepar, $tsLab_2, $tsCbo_2, $tsBRead))
   #
   #
   $tsLab_1.Text = "Pages:"
   #
   #
   $tsCbo_1.Items.AddRange(@('Cmdlet', 'About'))
   $tsCbo_1.SelectedIndex = 0
   $tsCbo_1.Add_SelectedIndexChanged($tsCbo_1_SelectedIndexChanged)
   #
   #
   $tsLab_2.Text = "Item:"
   #
   #
   $tsCbo_2.Size = New-Object Drawing.Size(377, 19)
   #
   #
   $tsBRead.Text = "Read"
   $tsBRead.Add_Click($tsBRead_Click)
   #
   #
   $txtRead.BackColor = [Drawing.Color]::DarkBlue
   $txtRead.Font = New-Object Drawing.Font('Tahoma', 10, [Drawing.FontStyle]::Bold)
   $txtRead.ForeColor = [Drawing.Color]::LightGray
   $txtRead.Dock = "Fill"
   $txtRead.Multiline = $true
   $txtRead.ReadOnly = $true
   $txtRead.ScrollBars = "Vertical"
   #
   #
   $sbPanel.Font = New-Object Drawing.Font('Tahoma', 8, [Drawing.FontStyle]::Italic)
   $sbPanel.SizingGrip = $false
   $sbPanel.Text = "Follow me on Twitter @gregzakharov"
   #
   #
   $frmMain.ClientSize = New-Object Drawing.Size(630, 370)
   $frmMain.Controls.AddRange(@($txtRead, $sbPanel, $tsTools, $mnuMain))
   $frmMain.FormBorderStyle = "FixedSingle"
   $frmMain.Icon = $ico
   $frmMain.StartPosition = "CenterScreen"
   $frmMain.Text = "Quick Guide"
   $frmMain.Add_Load({Add-CmdletPages})
   
   [void]$frmMain.ShowDialog()
 }
 
 frmMain_Show
`

