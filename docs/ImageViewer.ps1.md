---
Author: kakto_oz
Publisher: 
Copyright: 
Email: 
Version: 1.01
Encoding: ascii
License: cc0
PoshCode ID: 4848
Published Date: 2014-01-29t10
Archived Date: 2014-04-10t15
---

# imageviewer - 

## Description

this is not my script, author is greg zakharov. i found this script very useful for me.

## Comments



## Usage



## TODO



## function

``

## Code

`#
 #
 function frmMain_Show {
   Add-Type -AssemblyName System.Windows.Forms
   [Windows.Forms.Application]::EnableVisualStyles()
   
   $ico = [Drawing.Icon]::ExtractAssociatedIcon(($PSHome + '\powershell.exe'))
   
   $ia = New-Object Drawing.Imaging.ImageAttributes
   
   function getset([Drawing.Bitmap]$b) {
     $gfx = [Drawing.Graphics]::FromImage($b)
     $gfx.DrawImage($b, (New-Object Drawing.Rectangle 0, 0, $b.Width, $b.Height),
             0, 0, $b.Width, $b.Height, [Drawing.GraphicsUnit]::Pixel, $ia
     )
     $gfx.Flush()
   }
   
   $frmMain = New-Object Windows.Forms.Form
   $mnuMain = New-Object Windows.Forms.MenuStrip
   $mnuFile = New-Object Windows.Forms.ToolStripMenuItem
   $mnuOpen = New-Object Windows.Forms.ToolStripMenuItem
   $mnuExit = New-Object Windows.Forms.ToolStripMenuItem
   $mnuView = New-Object Windows.Forms.ToolStripMenuItem
   $mnuTool = New-Object Windows.Forms.ToolStripMenuItem
   $mnuHelp = New-Object Windows.Forms.ToolStripMenuItem
   $mnuInfo = New-Object Windows.Forms.ToolStripMenuItem
   $tsStrip = New-Object Windows.Forms.ToolStrip
   $tsLbl_1 = New-Object Windows.Forms.ToolStripLabel
   $tsLbl_2 = New-Object Windows.Forms.ToolStripLabel
   $tsLbl_3 = New-Object Windows.Forms.ToolStripLabel
   $tsCbo_1 = New-Object Windows.Forms.ToolStripComboBox
   $tsCbo_2 = New-Object Windows.Forms.ToolStripComboBox
   $tsBtn_1 = New-Object Windows.Forms.ToolStripButton
   $tsBtn_2 = New-Object Windows.Forms.ToolStripButton
   $pbImage = New-Object Windows.Forms.PictureBox
   $sbStrip = New-Object Windows.Forms.StatusStrip
   $sbLabel = New-Object Windows.Forms.ToolStripStatusLabel
   #
   #
   $mnuMain.Items.AddRange(@($mnuFile, $mnuView, $mnuHelp))
   $tsLbl_1.Text = "Mode:"
   $tsLbl_2.Text = "Style:"
   $tsLbl_3.Text = "Rotation:"
   $pbImage.Dock = [Windows.Forms.DockStyle]::Fill
   $sbLabel.AutoSize = $true
   #
   #
   $mnuFile.DropDownItems.AddRange(@($mnuOpen, $mnuExit))
   $mnuFile.Text = "&File"
   #
   #
   $mnuOpen.ShortcutKeys = [Windows.Forms.Keys]::Control, [Windows.Forms.Keys]::O
   $mnuOpen.Text = "&Open..."
   $mnuOpen.Add_Click({
     
     (New-Object Windows.Forms.OpenFileDialog) | % {
       $_.Filter = "JPEG (*.jpg;*.jpeg)|*.jpg;*.jpeg|GIF (*.gif)|*.gif|PNG (*.png)|*.png"
       $_.InitialDirectory = $pwd.Path
       
       if ($_.ShowDialog() -eq [Windows.Forms.DialogResult]::OK) {
         $img = [Drawing.Image]::FromFile($_.FileName)
         $pbImage.Image = $img
         $pbImage.SizeMode = [Windows.Forms.PictureBoxSizeMode]$tsCbo_1.SelectedItem
         $tsCbo_2.Enabled = $true
         $bmp = New-Object Drawing.Bitmap $img
       }
     }
     $sbLabel.Text = ("Width: {0}$(' ' * 3)Height: {1}" -f $img.Width, $img.Height)
   })
   #
   #
   $mnuExit.ShortcutKeys = [Windows.Forms.Keys]::Control, [Windows.Forms.Keys]::X
   $mnuExit.Text = "E&xit"
   $mnuExit.Add_Click({$frmMain.Close()})
   #
   #
   $mnuView.DropDownItems.AddRange(@($mnuTool))
   $mnuView.Text = "&View"
   #
   #
   $mnuTool.ShortcutKeys = [Windows.Forms.Keys]::Control, [Windows.Forms.Keys]::T
   $mnuTool.Text = "&Tools Panel"
   $mnuTool.Add_Click({
     $toggle =! $mnuTool.Checked
     $mnuTool.Checked = $toggle
     $tsStrip.Visible = $toggle
   })
   #
   #
   $mnuHelp.DropDownItems.AddRange(@($mnuInfo))
   $mnuHelp.Text = "&Help"
   #
   #
   $mnuInfo.Text = "About..."
   $mnuInfo.Add_Click({frmInfo_Show})
   #
   #
   $tsStrip.Items.AddRange(@($tsLbl_1, $tsCbo_1, $tsLbl_2, $tsCbo_2, $tsLbl_3, $tsBtn_1, $tsBtn_2))
   $tsStrip.Visible = $false
   #
   #
   [Enum]::GetValues([Windows.Forms.PictureBoxSizeMode]) | % {
     [void]$tsCbo_1.Items.Add($_)
   }
   $tsCbo_1.SelectedIndex = 4
   $tsCbo_1.Add_SelectedIndexChanged({
     $pbImage.SizeMode = [Windows.Forms.PictureBoxSizeMode]$tsCbo_1.SelectedItem
   })
   #
   #
   $tsCbo_2.Enabled = $false
   $tsCbo_2.Items.AddRange(@('Normal', 'Negative', 'Retro'))
   $tsCbo_2.SelectedIndex = 0
   $tsCbo_2.Add_SelectedIndexChanged({
     switch ($tsCbo_2.SelectedIndex) {
       0 {$pbImage.Image = $def}
       1 {
         if ($neg -eq $null) {
           $cm = New-Object Drawing.Imaging.ColorMatrix
           $cm.Matrix40 = $cm.Matrix41 = $cm.Matrix42 = 1
           $cm.Matrix00 = $cm.Matrix11 = $cm.Matrix22 = -1
           $ia.SetColorMatrix($cm)
           
           getset $bmp
           
           $neg = $bmp.Clone()
         }
         $pbImage.Image = $neg
       }
       2 {
         $bmp = $def.Clone()
         if ($ret -eq $null) {
           $cm = New-Object Drawing.Imaging.ColorMatrix
           $cm.Matrix00 = $cm.Matrix01 = $cm.Matrix02 = $cm.Matrix10 = $cm.Matrix11 = `
           $cm.Matrix12 = $cm.Matrix20 = $cm.Matrix21 = $cm.Matrix22 = 1/3
           $ia.SetColorMatrix($cm)
           
           getset $bmp
           
           $ret = $bmp.Clone()
         }
         $pbImage.Image = $ret
       }
   })
   #
   #
   $tsBtn_1.Text = "Left"
   $tsBtn_1.Add_Click({
     if ($pbImage.Image -ne $null) {
       $pbImage.Image.RotateFlip([Drawing.RotateFlipType]::Rotate90FlipXY)
       $pbImage.Refresh()
     }
   })
   #
   #
   $tsBtn_2.Text = "Right"
   $tsBtn_2.Add_Click({
     if ($pbImage.Image -ne $null) {
       $pbImage.Image.RotateFlip([Drawing.RotateFlipType]::Rotate270FlipXY)
       $pbImage.Refresh()
     }
   })
   #
   #
   $sbStrip.Items.AddRange(@($sbLabel))
   $sbStrip.SizingGrip = $false
   #
   #
   $frmMain.ClientSize = New-Object Drawing.Size(800, 547)
   $frmMain.Controls.AddRange(@($pbImage, $sbStrip, $tsStrip, $mnuMain))
   $frmMain.FormBorderStyle = [Windows.Forms.FormBorderStyle]::FixedSingle
   $frmMain.Icon = $ico
   $frmMain.MainMenuStrip = $mnuMain
   $frmMain.StartPosition = [Windows.Forms.FormStartPosition]::CenterScreen
   $frmMain.Text = "Image Viewer"
   $frmMain.Add_Load({$sbLabel.Text = "Ready"})
   
   [void]$frmMain.ShowDialog()
 }
 
 function frmInfo_Show {
   $frmInfo = New-Object Windows.Forms.Form
   $pbImage = New-Object Windows.Forms.PictureBox
   $lblName = New-Object Windows.Forms.Label
   $lblCopy = New-Object Windows.Forms.Label
   $btnExit = New-Object Windows.Forms.Button
   #
   #
   $pbImage.Image = $ico.ToBitmap()
   $pbImage.Location = New-Object Drawing.Point(16, 16)
   $pbImage.Size = New-Object Drawing.Size(32, 32)
   $pbImage.SizeMode = [Windows.Forms.PictureBoxSizeMode]::StretchImage
   #
   #
   $lblName.Font = $bol2
   $lblName.Location = New-Object Drawing.Point(53, 19)
   $lblName.Size = New-Object Drawing.Size(360, 18)
   $lblName.Text = "Image Viewer v1.01"
   #
   #
   $lblCopy.Location = New-Object Drawing.Point(67, 37)
   $lblCopy.Size = New-Object Drawing.Size(360, 23)
   $lblCopy.Text = "Copyright (C) 2013-2014 greg zakharov"
   #
   #
   $btnExit.Location = New-Object Drawing.Point(135, 67)
   $btnExit.Text = "OK"
   #
   #
   $frmInfo.AcceptButton = $btnExit
   $frmInfo.CancelButton = $btnExit
   $frmInfo.ClientSize = New-Object Drawing.Size(350, 110)
   $frmInfo.ControlBox = $false
   $frmInfo.Controls.AddRange(@($pbImage, $lblName, $lblCopy, $btnExit))
   $frmInfo.FormBorderStyle = [Windows.Forms.FormBorderStyle]::FixedSingle
   $frmInfo.ShowInTaskBar = $false
   $frmInfo.StartPosition = [Windows.Forms.FormStartPosition]::CenterParent
   $frmInfo.Text = "About..."
 
   [void]$frmInfo.ShowDialog()
 }
 
 frmMain_Show
`

