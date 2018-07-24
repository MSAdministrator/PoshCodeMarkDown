---
Author: silvia
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6807
Published Date: 2017-03-20t18
Archived Date: 2017-03-25t17
---

# tic tac toe demo - 

## Description

update $cur variable scope to $script

## Comments



## Usage



## TODO



## function

``

## Code

`#
 #
 function frmMain_Show {
   Add-Type -Assembly System.Windows.Forms
   [Windows.Forms.Application]::EnableVisualStyles()
   
   $ico = [Drawing.Icon]::ExtractAssociatedIcon(($PSHome + '\powershell.exe'))
 
   $btn = New-Object "Windows.Forms.Button[,]" 3, 3
   
   $frmMain = New-Object Windows.Forms.Form
   $btnPlay = New-Object Windows.Forms.Button
   #
   #
   $btnPlay.Location = New-Object Drawing.Point(78, 225)
   $btnPlay.Text = "New Play"
   $btnPlay.Add_Click({
     $script:cur = $false
     $btn | % {$_.Text = [String]::Empty}
   })
   #
   #
   $frmMain.ClientSize = New-Object Drawing.Size(231, 260)
   $frmMain.Controls.Add($btnPlay)
   $frmMain.FormBorderStyle = [Windows.Forms.FormBorderStyle]::FixedSingle
   $frmMain.Icon = $ico
   $frmMain.MainMenuStrip = $mnuMain
   $frmMain.MaximizeBox = $false
   $frmMain.MinimizeBox = $False
   $frmmain.StartPosition = [Windows.Forms.FormStartPosition]::CenterScreen
   $frmMain.Text = "Tic Tac Toe"
   $frmMain.Add_Load({
     for ($i = 0; $i -lt 3; $i++) {
       for ($j = 0; $j -lt 3; $j++) {
         $btn[$i, $j] = New-Object Windows.Forms.Button
         $btn[$i, $j].Parent = $this
         $btn[$i, $j].Left = 10 + $j * 70
         $btn[$i, $j].Top = 10 + $i * 70
         $btn[$i, $j].Size = New-Object Drawing.Size(70, 70)
         $btn[$i, $j].Font = New-Object Drawing.Font("Microsoft Sans Serif", 27, [Drawing.FontStyle]::Bold)
         $btn[$i, $j].Add_Click({
           if ([String]::IsNullOrEmpty($this.Text)) {
             switch ($script:cur) {
               $true {
                 $this.ForeColor = [Drawing.Color]::Crimson
                 $this.Text = "O"
                 $script:cur = $false
               }
               default {
                 $this.ForeColor = [Drawing.Color]::DarkGreen
                 $this.Text = "X"
                 $script:cur = $true
               }
           }
   })
   
   [void]$frmmain.ShowDialog()
 }
 
 frmMain_Show
`

