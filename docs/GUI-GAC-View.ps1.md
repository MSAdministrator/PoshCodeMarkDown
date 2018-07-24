---
Author: greg zakharov
Publisher: 
Copyright: 
Email: 
Version: 1.01
Encoding: ascii
License: cc0
PoshCode ID: 5088
Published Date: 2014-04-18t09
Archived Date: 2014-04-21t08
---

# gui gac view - 

## Description

fixed

## Comments



## Usage



## TODO



## function

`get-itemslist`

## Code

`#
 #
 function Get-ItemsList {
   $lvItems.Items.Clear()
   [IO.Directory]::GetFiles(([Regex]"(?<=file:///)(.*)(?=GAC)").Match(
     ([AppDomain]::CurrentDomain.GetAssemblies()[0].Evidence | ? {
       $_.Value -ne $null
   }).Value), '*.dll', [IO.SearchOption]::AllDirectories) | % {$tbl = @{}}{
     try {
       $ra = [Reflection.AssemblyName]::GetAssemblyName($_)
       $tbl[$ra.FullName] = $ra
     }
     catch {}
   }{
     $tbl.GetEnumerator() | % {
       $asm = $_.Name.Split(',')
       $itm = $lvItems.Items.Add($asm[0], 0)
       $itm.SubItems.Add($asm[1].Split('=')[1])
       $itm.SubItems.Add($asm[2].Split('=')[1])
       $itm.SubItems.Add($asm[3].Split('=')[1])
       $itm.SubItems.Add($_.Value.ProcessorArchitecture.ToString())
       $itm.SubItems.Add($_.Value.CodeBase)
     }
   }
   $lvItems.AutoResizeColumns([Windows.Forms.ColumnHeaderAutoResizeStyle]::ColumnContent)
   Get-ItemsNumber
 }
 
 function Get-ItemsNumber {
   $sbLabel.Text = $lvItems.Items.Count.ToString() + ' item(s)'
 }
 
 function frmMain_Show {
   Add-Type -AssemblyName System.Windows.Forms
   [Windows.Forms.Application]::EnableVisualStyles()
   
   $ico = [Drawing.Icon]::ExtractAssociatedIcon(($PSHome + '\powershell.exe'))
   
   $frmMain = New-Object Windows.Forms.Form
   $mnuMain = New-Object Windows.Forms.MenuStrip
   $mnuFile = New-Object Windows.Forms.ToolStripMenuItem
   $mnuScan = New-Object Windows.Forms.ToolStripMenuItem
   $mnuExit = New-Object Windows.Forms.ToolStripMenuItem
   $mnuView = New-Object Windows.Forms.ToolStripMenuItem
   $mnuHelp = New-Object Windows.Forms.ToolStripMenuItem
   $mnuInfo = New-Object Windows.Forms.ToolStripMenuItem
   $lvItems = New-Object Windows.Forms.ListView
   $chCol_1 = New-Object Windows.Forms.ColumnHeader
   $chCol_2 = New-Object Windows.Forms.ColumnHeader
   $chCol_3 = New-Object Windows.Forms.ColumnHeader
   $chCol_4 = New-Object Windows.Forms.ColumnHeader
   $chCol_5 = New-Object Windows.Forms.ColumnHeader
   $chCol_6 = New-Object Windows.Forms.ColumnHeader
   $imgList = New-Object Windows.Forms.ImageList
   $sbStrip = New-Object Windows.Forms.StatusStrip
   $sbLabel = New-Object Windows.Forms.ToolStripStatusLabel
   #
   #
   $mnuMain.Items.AddRange(@($mnuFile, $mnuHelp))
   $chCol_1.Text = "Assembly Name"
   $chCol_2.Text = "Version"
   $chCol_3.Text = "Culture"
   $chCol_4.Text = "Public Key Token"
   $chCol_5.Text = "Kind"
   $chCol_6.Text = "Full Path"
   $sbStrip.Items.AddRange(@($sbLabel))
   $sbLabel.AutoSize = $true
   $imgList.Images.Add($ico.ToBitmap())
   #
   #
   $mnuFile.DropDownItems.AddRange(@($mnuScan, $mnuExit))
   $mnuFile.Text = "&File"
   #
   #
   $mnuScan.ShortcutKeys = [Windows.Forms.Keys]::F5
   $mnuScan.Text = "&Scan"
   $mnuScan.Add_Click({Get-ItemsList})
   #
   #
   $mnuExit.ShortcutKeys = [Windows.Forms.Keys]::Control, [Windows.Forms.Keys]::X
   $mnuExit.Text = "E&xit"
   $mnuExit.Add_Click({$frmMain.Close()})
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
   $lvItems.AllowColumnReorder = $true
   $lvItems.Columns.AddRange(@($chCol_1, $chCol_2, $chCol_3, $chCol_4, $chCol_5, $chCol_6))
   $lvItems.Dock = [Windows.Forms.DockStyle]::Fill
   $lvItems.SmallImageList = $imgList
   $lvItems.Sorting = [Windows.Forms.SortOrder]::Ascending
   $lvItems.View = [Windows.Forms.View]::Details
   #
   #
   $frmMain.ClientSize = New-Object Drawing.Size(570, 320)
   $frmMain.Controls.AddRange(@($lvItems, $sbStrip, $mnuMain))
   $frmMain.Icon = $ico
   $frmMain.MainMenuStrip = $mnuMain
   $frmMain.StartPosition = [Windows.Forms.FormStartPosition]::CenterScreen
   $frmMain.Text = "GACView"
   $frmMain.Add_Load({Get-ItemsNumber})
   
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
   $lblName.Font = New-Object Drawing.Font("Microsoft Sans Serif", 8.5, [Drawing.FontStyle]::Bold)
   $lblName.Location = New-Object Drawing.Point(53, 19)
   $lblName.Size = New-Object Drawing.Size(360, 18)
   $lblName.Text = "GACView v1.01"
   #
   #
   $lblCopy.Location = New-Object Drawing.Point(67, 37)
   $lblCopy.Size = New-Object Drawing.Size(360, 23)
   $lblCopy.Text = "Copyright (C) 2014 greg zakharov"
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
   $frmInfo.Add_Load($frmInfo_Load)
 
   [void]$frmInfo.ShowDialog()
 }
 
 frmMain_Show
`

