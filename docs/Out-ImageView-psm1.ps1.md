---
Author: greg zakharov
Publisher: 
Copyright: 
Email: 
Version: 1.00
Encoding: ascii
License: cc0
PoshCode ID: 5162
Published Date: 2014-05-16t13
Archived Date: 2014-08-29t23
---

# out-imageview.psm1 - 

## Description

screenshot at http

## Comments



## Usage



## TODO



## function

`out-imageview`

## Code

`#
 #
 Set-Alias oiv Out-ImageView
 
 function Out-ImageView {
   <#
     .NOTES
         Author : gregzakharov
         Version: 1.00
   #>
   param(
     [Parameter(Mandatory=$true, ValueFromPipeline=$true)]
     [ValidateScript({Test-Path $_})]
     [String]$Image
   )
   
   begin {
     $Image = cvpa $Image
     
     function frmMain_Show($Image) {
       Add-Type -AssemblyName System.Windows.Forms
       [Windows.Forms.Application]::EnableVisualStyles()
       
       $ico = [Drawing.Icon]::ExtractAssociatedIcon(($PSHome + '\powershell.exe'))
       $undo = "iVBORw0KGgoAAAANSUhEUgAAABAAAAAQCAIAAACQkWg2AAAAAXNSR0IArs4c6QAAAARnQU1BAACxjwv8YQUAAAAgY0hSTQAAeiYAAICEA" + `
               "AD6AAAAgOgAAHUwAADqYAAAOpgAABdwnLpRPAAAAFdJREFUOE+V0cEKACAIA9D6/2MfvCShYoI68SR7CDoBDKkMUK0VZ28yYloAFvVO6m" + `
               "246Rb40zWgdAFiOp/YC+wPSp/HcddXIlMD2tMCv+mCawTgRgNZHNgzVWdfZ6a/3QAAAABJRU5ErkJggg=="
       $redo = "iVBORw0KGgoAAAANSUhEUgAAABAAAAAQCAIAAACQkWg2AAAABGdBTUEAALGPC/xhBQAAAFdJREFUOE+V0dEOACAEBVD9/6MPltbWDHGZr" + `
               "Yc6JpaI0CgU/II5uaEaRNMDZyBgDQqeGYBrSI9R6gp0D5MsxhoLnZZwcF+i4NWFgO2iB67nBsQfViCdxwafG2df4Xe7AgAAAABJRU5Erk" + `
               "Jggg=="
       $ia = New-Object Drawing.Imaging.ImageAttributes
       function Get-Image([String]$i) {
         [Drawing.Image]::FromStream(
           (New-Object IO.MemoryStream(($$ = [Convert]::FromBase64String($i)), 0, $$.Length))
         )
       }
       
       function Set-Style([Drawing.Bitmap]$b) {
         $g = [Drawing.Graphics]::FromImage($b)
         $g.DrawImage($b, (New-Object Drawing.Rectangle 0, 0, $b.Width, $b.Height),
                     0, 0, $b.Width, $b.Height, [Drawing.GraphicsUnit]::Pixel, $ia)
         $g.Flush()
       }
       
       $frmMain = New-Object Windows.Forms.Form
       $tsStrip = New-Object Windows.Forms.ToolStrip
       $tsLbl_1 = New-Object Windows.Forms.ToolStripLabel
       $tsLbl_2 = New-Object Windows.Forms.ToolStripLabel
       $tsLbl_3 = New-Object Windows.Forms.ToolStripLabel
       $tsBtn_1 = New-Object Windows.Forms.ToolStripButton
       $tsBtn_2 = New-Object Windows.Forms.ToolStripButton
       $tsCbo_1 = New-Object Windows.Forms.ToolStripComboBox
       $tsCbo_2 = New-Object Windows.Forms.ToolStripComboBox
       $pbImage = New-Object Windows.Forms.PictureBox
       $sbStrip = New-Object Windows.Forms.StatusStrip
       $sbLabel = New-Object Windows.Forms.ToolStripStatusLabel
       #
       #
       $tsStrip.Items.AddRange(@($tsLbl_1, $tsBtn_1, $tsBtn_2, $tsLbl_2, $tsCbo_1, $tsLbl_3, $tsCbo_2))
       $tsLbl_1.Text = "Rotation:"
       $tsLbl_2.Text = " Mode:"
       $tsLbl_3.Text = " Style:"
       $sbStrip.Items.AddRange(@($sbLabel))
       $sbLabel.AutoSize = $true
       #
       #
       $tsBtn_1.Image = Get-Image $undo
       $tsBtn_1.Add_Click({
         $pbImage.Image.RotateFlip([Drawing.RotateFlipType]::Rotate90FlipXY)
         $pbImage.Refresh()
       })
       #
       #
       $tsBtn_2.Image = Get-Image $redo
       $tsBtn_2.Add_Click({
         $pbImage.Image.RotateFlip([Drawing.RotateFlipType]::Rotate270FlipXY)
         $pbImage.Refresh()
       })
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
       $tsCbo_2.Items.AddRange(@('Normal', 'Negative', 'Retro'))
       $tsCbo_2.SelectedIndex = 0
       $tsCbo_2.Add_SelectedIndexChanged({
         switch ($tsCbo_2.SelectedIndex) {
           0 { $pbImage.Image = $def }
           1 {
             if ($neg -eq $null) {
               $cm = New-Object Drawing.Imaging.ColorMatrix
               $cm.Matrix40 = $cm.Matrix41 = $cm.Matrix42 = 1
               $cm.Matrix00 = $cm.Matrix11 = $cm.Matrix22 = -1
               $ia.SetColorMatrix($cm)
               
               Set-Style $bmp
               $neg = $bmp.Clone()
             $pbImage.Image = $neg
           }
           2 {
             $bmp = $def.Clone()
             if ($ret -eq $null) {
               $cm = New-Object Drawing.Imaging.ColorMatrix
               $cm.Matrix00 = $cm.Matrix01 = $cm.Matrix02 = $cm.Matrix10 = $cm.Matrix11 = `
               $cm.Matrix12 = $cm.Matrix20 = $cm.Matrix21 = $cm.Matrix22 = 1/3
               $ia.SetColorMatrix($cm)
               
               Set-Style $bmp
               $ret = $bmp.Clone()
             $pbImage.Image = $ret
           }
       })
       #
       #
       $pbImage.BackColor = [Drawing.Color]::DarkGray
       $pbImage.Dock = [Windows.Forms.DockStyle]::Fill
       #
       #
       $frmMain.ClientSize = New-Object Drawing.Size(800, 547)
       $frmMain.Controls.AddRange(@($pbImage, $sbStrip, $tsStrip))
       $frmMain.Icon = $ico
       $frmMain.StartPosition = [Windows.Forms.FormStartPosition]::CenterScreen
       $frmMain.Text = $PSCmdlet.CommandRuntime
       $frmMain.Add_Load({
         try {
           $neg = $ret = $null
           $img = [Drawing.Image]::FromFile($Image)
           $pbImage.Image = $img
           $pbImage.SizeMode = [Windows.Forms.PictureBoxSizeMode]$tsCbo_1.SelectedItem
           $bmp = New-Object Drawing.Bitmap $img
           $def = $bmp.Clone()
           
           $sbLabel.Text = ("Width: {0}$(' ' * 3)Height: {1}" -f $img.Width, $img.Height)
         }
         catch [Management.Automation.MethodInvocationException] { [Windows.Forms.Application]::Exit() }
       })
       
       [void]$frmMain.ShowDialog()
   }
   process {
     frmMain_Show $Image
   }
   end {}
 }
 
 Export-ModuleMember -Alias oiv -Function Out-ImageView
`

