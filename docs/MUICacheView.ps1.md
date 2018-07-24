---
Author: greg zakharov
Publisher: 
Copyright: 
Email: 
Version: 1.03
Encoding: ascii
License: cc0
PoshCode ID: 4428
Published Date: 2013-08-31t13
Archived Date: 2013-09-03t04
---

# muicacheview - 

## Description

what is muicache and why it necessary to ask your favorite search engine, for example, google, bing and etc. i just updated version of my old script which is analog of eponymous nir sofer’s tool (hi, nir! how are you?) in this version

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 $dir = (gci $MyInvocation.MyCommand.Name).Directory
 $key = "Software\Microsoft\Windows\ShellNoRoam\MUICache"
 $csv = (Split-Path (gci $MyInvocation.InvocationName).FullName) + "\MUICacheView.csv"
 
 function ItemsCounting {
   $tlStrip.Text = $lstView.Items.Count.ToString() + " item(s)"
 }
 
 function BuildItemsList([string]$nod) {
   $rk = [Microsoft.Win32.Registry]::CurrentUser.OpenSubKey($key)
   $itm = $lstView.Items.Add($nod, $i)
   $itm.SubItems.Add($rk.GetValue($nod).ToString())
   $rk.Close()
 }
 
 function InvokeScaning {
   $lstView.Items.Clear()
   $imgList.Images.Clear()
   
   [int]$i = 0
   $rk = [Microsoft.Win32.Registry]::CurrentUser.OpenSubKey($key)
   $rk.GetValueNames() | % {
     if ($rk.GetValueKind($_).ToString() -ne "Binary") {
       if ($mnuHide.Checked) {
         if (-not ($_.StartsWith("@"))) {
           BuildItemsList($_)
           ++$i
           $imgList.Images.Add([Drawing.Icon]::ExtractAssociatedIcon($_).ToBitmap())
         }
       }
       else {
         BuildItemsList($_)
         if (-not ($_.StartsWith("@"))) {
           ++$i
           $imgList.Images.Add([Drawing.Icon]::ExtractAssociatedIcon($_).ToBitmap())
         } else {
           $sub = $_.Substring(1, $_.IndexOf(",") - 1)
           if ($sub -imatch "%\w+%") {$sub = $sub -replace "%\w+%", "$env:systemroot"}
           if ($sub -match "^explorer.exe$") {$sub = $env:windir + "\" + $sub}
           if ($sub -match "^\w+.dll$") {$sub = [Environment]::SystemDirectory + "\" + $sub}
           ++$i
           $imgList.Images.Add([Drawing.Icon]::ExtractAssociatedIcon($sub).ToBitmap())
         }
   }
   $rk.Close()
 }
 
 function ChangeLanguage([string]$loc) {
   switch ($loc) {
     "en" {
       $mnuIEng.Checked = $true
       $mnuIRus.Checked = $false
       
       $mnuFile.Text = "&File"
       $mnuScan.Text = "S&can"
       $mnuSave.Text = "&Save"
       $mnuExit.Text = "E&xit"
       $mnuEdit.Text = "&Edit"
       $mnuKill.Text = "&Delete Item"
       $mnuView.Text = "&View"
       $mnuHide.Text = "&Hide System Entries"
       $mnuSBar.Text = "Show Status &Bar"
       $mnuFont.Text = "&Font..."
       $mnuLang.Text = "&Language"
       $mnuIEng.Text = "English"
       $mnuIRus.Text = "Russian"
       $mnuHelp.Text = "&Help"
       $mnuInfo.Text = "About"
       $chPath_.Text = "Application Path"
       $chDesc_.Text = "Application Description"
     "ru" {
       $mnuIEng.Checked = $false
       $mnuIRus.Checked = $true
       
   }
 }
 
 $mnuSave_Click= {
   if ($lstView.Items.Count -ne 0) {
     (New-Object Windows.Forms.SaveFileDialog) | % {
       $_.FileName = "MUICacheView"
       $_.Filter = "CSV (*.csv)|*.csv"
       $_.InitialDirectory = $dir
       
       if ($_.ShowDialog() -eq [Windows.Forms.DialogResult]::OK) {
         $sw = New-Object IO.StreamWriter($_.FileName, [Text.Encoding]::Default)
         $sw.WriteLine("Application Path, Application Description")
         $lstView.Items | % {
           $sw.WriteLine($($_.Text + ', ' + $_.SubItems[1].Text))
         }
         $sw.Flush()
         $sw.Close()
       }
 }
 
 $mnuKIll_Click= {
   $rk = [Microsoft.Win32.Registry]::CurrentUser.OpenSubKey($key, $true)
   
   for ([int]$i = 0; $i -lt $lstView.Items.Count; $i++) {
     if ($lstView.Items[$i].Selected) {
       $rk.DeleteValue($lstView.Items[$i].Text, $false)
       $lstView.Items[$i].Remove()
       $i--
     }
   }
   
   ItemsCounting
 }
 
 $mnuHide_Click= {
   [bool]$tgl =! $mnuHide.Checked
   $mnuHide.Checked = $tgl
   
   InvokeScaning
   ItemsCounting
 }
 
 $mnuSBar_Click= {
   [bool]$tgl =! $mnuSBar.Checked
   $mnuSBar.Checked = $tgl
   $sbInfo_.Visible = $tgl
 }
 
 $mnuFont_Click= {
   (New-Object Windows.Forms.FontDialog) | % {
     $_.MaxSize = 12
     $_.ShowEffects = $false
     
     if ($_.ShowDialog() -eq [Windows.Forms.DialogResult]::OK) {
       $lstView.Font = $_.Font
 }
 
 function frmMain_Show {
   Add-Type -Assembly System.Windows.Forms
   [Windows.Forms.Application]::EnableVisualStyles()
   
   $ico = [Drawing.Icon]::ExtractAssociatedIcon($pshome + "\powershell.exe")
   
   $frmMain = New-Object Windows.Forms.Form
   $mnuMain = New-Object Windows.Forms.MenuStrip
   $mnuFile = New-Object Windows.Forms.ToolStripMenuItem
   $mnuScan = New-Object Windows.Forms.ToolStripMenuItem
   $mnuSave = New-Object Windows.Forms.ToolStripMenuItem
   $mnuNul1 = New-Object Windows.Forms.ToolStripSeparator
   $mnuExit = New-Object Windows.Forms.ToolStripMenuItem
   $mnuEdit = New-Object Windows.Forms.ToolStripMenuItem
   $mnuKill = New-Object Windows.Forms.ToolStripMenuItem
   $mnuView = New-Object Windows.Forms.ToolStripMenuItem
   $mnuHide = New-Object Windows.Forms.ToolStripMenuItem
   $mnuSBar = New-Object Windows.Forms.ToolStripMenuItem
   $mnuNul2 = New-Object Windows.Forms.ToolStripSeparator
   $mnuFont = New-Object Windows.Forms.ToolStripMenuItem
   $mnuNul3 = New-Object Windows.Forms.ToolStripSeparator
   $mnuLang = New-Object Windows.Forms.ToolStripMenuItem
   $mnuIEng = New-Object Windows.Forms.ToolStripMenuItem
   $mnuIRus = New-Object Windows.Forms.ToolStripMenuItem
   $mnuHelp = New-Object Windows.Forms.ToolStripMenuItem
   $mnuInfo = New-Object Windows.Forms.ToolStripMenuItem
   $lstView = New-Object Windows.Forms.ListView
   $imgList = New-Object Windows.Forms.ImageList
   $chPath_ = New-Object Windows.Forms.ColumnHeader
   $chDesc_ = New-Object Windows.Forms.ColumnHeader
   $stStrip = New-Object Windows.Forms.StatusStrip
   $tlStrip = New-Object Windows.Forms.ToolStripStatusLabel
   #
   #
   $mnuMain.Items.AddRange(@($mnuFile, $mnuEdit, $mnuView, $mnuHelp))
   #
   #
   $mnuFile.DropDownItems.AddRange(@($mnuScan, $mnuSave, $mnuNul1, $mnuExit))
   #
   #
   $mnuScan.ShortcutKeys = "F5"
   $mnuScan.Add_Click({InvokeScaning;ItemsCounting})
   #
   #
   $mnuSave.ShortcutKeys = "Control, S"
   $mnuSave.Add_Click($mnuSave_Click)
   #
   #
   $mnuExit.ShortcutKeys = "Control, X"
   $mnuExit.Add_Click({$frmMain.Close()})
   #
   #
   $mnuEdit.DropDownItems.AddRange(@($mnuKill))
   #
   #
   $mnuKill.ShortcutKeys = "Del"
   $mnuKill.Add_Click($mnuKill_Click)
   #
   #
   $mnuView.DropDownItems.AddRange(@($mnuHide, $mnuSBar, $mnuNul2, $mnuFont, $mnuNul3, $mnuLang))
   #
   #
   $mnuHide.Checked = $true
   $mnuHide.ShortcutKeys = "Control, H"
   $mnuHide.Add_Click($mnuHide_Click)
   #
   #
   $mnuSBar.Checked = $true
   $mnuSBar.ShortcutKeys = "Control, B"
   $mnuSBar.Add_Click($mnuSBar_Click)
   #
   #
   $mnuFont.Add_Click($mnuFont_Click)
   #
   #
   $mnuLang.DropDownItems.AddRange(@($mnuIEng, $mnuIRus))
   #
   #
   $mnuIEng.Add_Click({ChangeLanguage("en")})
   #
   #
   $mnuIRus.Add_Click({ChangeLanguage("ru")})
   #
   #
   $mnuHelp.DropDownItems.AddRange(@($mnuInfo))
   #
   #
   $mnuInfo.Add_Click({frmAbout_Show})
   #
   #
   $lstView.AllowColumnReorder = $true
   $lstView.Columns.AddRange(@($chPath_, $chDesc_))
   $lstView.Dock = "Fill"
   $lstView.FullRowSelect = $true
   $lstView.MultiSelect = $false
   $lstView.SmallImageList = $imgList
   $lstView.Sorting = "Ascending"
   $lstView.View = "Details"
   #
   #
   $imgList.ImageSize = New-Object Drawing.Size(17, 15)
   #
   #
   $chPath_.Width = 275
   #
   #
   $chDesc_.Width = 330
   #
   #
   $stStrip.Items.AddRange(@($tlStrip))
   $stStrip.SizingGrip = $false
   #
   #
   $tlStrip.Alignment = "Left"
   $tlStrip.BorderStyle = "Raised"
   #
   #
   $frmMain.ClientSize = New-Object Drawing.Size(573, 217)
   $frmMain.Controls.AddRange(@($lstView, $stStrip, $mnuMain))
   $frmMain.MainMenuStrip = $mnuMain
   $frmMain.MaximizeBox = $false
   $frmMain.Icon = $ico
   $frmMain.StartPosition = "CenterScreen"
   $frmMain.Text = "MUICacheView"
   $frmMain.Add_Load({ItemsCounting; ChangeLanguage("en")})
   
   [void]$frmMain.ShowDialog()
 }
 
 function frmAbout_Show {
   $frmMain = New-Object Windows.Forms.Form
   $pbImage = New-Object Windows.Forms.PictureBox
   $lblName = New-Object Windows.Forms.Label
   $lblCopy = New-Object Windows.Forms.Label
   $btnExit = New-Object Windows.Forms.Button
   #
   #
   $pbImage.Location = New-Object Drawing.Point(16, 16)
   $pbImage.Size = New-Object Drawing.Size(32, 32)
   $pbImage.SizeMode = "StretchImage"
   #
   #
   $lblName.Font = New-Object Drawing.Font("Microsoft Sans Serif", 8.5, [Drawing.FontStyle]::Bold)
   $lblName.Location = New-Object Drawing.Point(53, 19)
   $lblName.Size = New-Object Drawing.Size(360, 18)
   $lblName.Text = "MUICacheView v1.03"
   #
   #
   $lblCopy.Location = New-Object Drawing.Point(67, 37)
   $lblCopy.Size = New-Object Drawing.Size(360, 15)
   $lblCopy.Text = "Copyright (C) 2012-2013 gregzakh@gmail.com"
   #
   #
   $btnExit.Location = New-Object Drawing.Point(135, 57)
   $btnExit.Text = "OK"
   #
   #
   $frmMain.AcceptButton = $btnExit
   $frmMain.CancelButton = $btnExit
   $frmMain.ClientSize = New-Object Drawing.Size(350, 90)
   $frmMain.ControlBox = $false
   $frmMain.Controls.AddRange(@($pbImage, $lblName, $lblCopy, $btnExit))
   $frmMain.FormBorderStyle = "FixedSingle"
   $frmMain.ShowInTaskbar = $false
   $frmMain.StartPosition = "CenterParent"
   $frmMain.Text = "About..."
   $frmMain.Add_Load({$pbImage.Image = $ico.ToBitmap()})
   
   [void]$frmMain.ShowDialog()
 }
 
 frmMain_Show
`

