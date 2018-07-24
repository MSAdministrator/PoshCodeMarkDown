---
Author: greg zakharov
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 4208
Published Date: 2013-06-19t16
Archived Date: 2013-06-25t00
---

# dropdown window - 

## Description

just launch script to see what’s about this script. press f12 to hide window it and show it again. use escape to close window.

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 $frmMain_KeyDown= {
   if ($_.KeyCode -eq "F12") {
     switch ($vis) {
       $true {
         $tmrTick.Enabled = $true
         while ($frmMain.Location.Y -ne -$frmMain.Height) {
           $frmMain.Top--
         }
         $tmrTick.Enabled = $false
         $script:vis = $false
         break
       }
 
       $false {
         Toggle; break
       }
     }
   }
   elseif ($_.KeyCode -eq "Escape") {
     $frmMain.Close()
   }
 }
 
 function Toggle {
   $tmrTick.Enabled = $true
   while ($frmMain.Location.Y -ne 0) {
     $frmMain.Top++
   }
   $tmrTick.Enabled = $false
   $script:vis = $true
 }
 
 function frmMain_Show {
   Add-Type -AssemblyName System.Windows.Forms
 
   $rec = [Windows.Forms.Screen]::PrimaryScreen.Bounds
 
   $frmMain = New-Object Windows.Forms.Form
   $tmrTick = New-Object Windows.Forms.Timer
   #
   #
   $tmrTick.Interval = 1000
   $tmrTick.Add_Tick({Toggle})
   #
   #
   $frmMain.BackColor = [Drawing.Color]::Black
   $frmMain.ClientSize = New-Object Drawing.Size($rec.Width, ($rec.Height / 2))
   $frmMain.FormBorderStyle = "None"
   $frmMain.Location = New-Object Drawing.Point(0, -$frmMain.Height)
   #$frmMain.Opacity = .5
   $frmMain.StartPosition = "Manual"
   $frmMain.Add_KeyDown($frmMain_KeyDown)
   $frmMain.Add_Load({$tmrTick.Enabled = $true})
 
   [void]$frmMain.ShowDialog()
 }
 
 frmMain_Show
`

