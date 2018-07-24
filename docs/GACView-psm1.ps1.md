---
Author: greg zakharov
Publisher: 
Copyright: 
Email: 
Version: 1.01
Encoding: ascii
License: cc0
PoshCode ID: 5193
Published Date: 2014-05-27t07
Archived Date: 2014-08-29t23
---

# gacview.psm1 - 

## Description

gac path extracts dynamically

## Comments



## Usage



## TODO



## function

`invoke-gacview`

## Code

`#
 #
 if (!(Test-Path alias:gacview)) { Set-Alias gacview Invoke-GACView }
 
 function Invoke-GACView {
   Add-Type -AssemblyName System.Windows.Forms
   [Windows.Forms.Application]::EnableVisualStyles()
   
   $ico = [Drawing.Icon]::ExtractAssociatedIcon(($PSHome + '\powershell.exe'))
   
   $img = "iVBORw0KGgoAAAANSUhEUgAAABIAAAASCAIAAADZrBkAAAAAAXNSR0IArs4c6QAAAARnQU1BAACxjwv8" + `
          "YQUAAAAgY0hSTQAAeiYAAICEAAD6AAAAgOgAAHUwAADqYAAAOpgAABdwnLpRPAAAAKpJREFUOE+9UlsO" + `
          "gCAMw4SD8+W5uBkWqgU2g9EP92FA1vUBWyklfCjAFrW38g1hjcFpzjnGaNoeYMCA7R0MDAB4TI3Di6Qf" + `
          "FWBmewMzGD+C5JWNWXGPNfywJG/802GKiwHgi3F0RYWtNXBQhw1nJxv7xN9EOdiYShNcmzT4uu5qZ2Iz" + `
          "3kg16nTb6wKoW944NaWTgYtnb+pQMBw6iTTeFLdf6EpWb3Lxyv+FHejVmVC66Eg0AAAAAElFTkSuQmCC"
   
   function Get-ItemsNumber {
     $sbLabel.Text = $lvItems.Items.Count.ToString() + " item(s)"
   }
   
   function Get-Set([String]$s) {
     return $s.Split('=')[1]
   }
   
   function Get-ResourceImage([String]$s) {
     [Drawing.Image]::FromStream(
       (New-Object IO.MemoryStream(($$ = [Convert]::FromBase64String($s)), 0, $$.Length))
     )
   }
   
   $frmMain = New-Object Windows.Forms.Form
   $mnuMain = New-Object Windows.Forms.MenuStrip
   $mnuFile = New-Object Windows.Forms.ToolStripMenuItem
   $mnuScan = New-Object Windows.Forms.ToolStripMenuItem
   $mnuNull = New-Object Windows.Forms.ToolStripSeparator
   $mnuExit = New-Object Windows.Forms.ToolStripMenuItem
   $mnuView = New-Object Windows.Forms.ToolStripMenuItem
   $mnuFont = New-Object Windows.Forms.ToolStripMenuItem
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
   $mnuMain.Items.AddRange(@($mnuFile, $mnuView, $mnuHelp))
   $chCol_1.Width = 190
   $chCol_2, $chCol_3 | % { $_.Width = 57 }
   $chCol_4.Width = 110
   $chCol_5.Width = 47
   $chCol_6.Width = 230
   $chCol_1.Text = "Assembly Name"
   $chCol_2.Text = "Version"
   $chCol_3.Text = "Culture"
   $chCol_4.Text = "Public Key Token"
   $chCol_5.Text = "Kind"
   $chCol_6.Text = "Path"
   $imgList.Images.Add((Get-ResourceImage $img))
   $sbStrip.Items.AddRange(@($sbLabel))
   $sbLabel.AutoSize = $true
   #
   #
   $mnuFile.DropDownItems.AddRange(@($mnuScan, $mnuNull, $mnuExit))
   $mnuFile.Text = "&File"
   #
   #
   $mnuScan.ShortcutKeys = [Windows.Forms.Keys]::F5
   $mnuScan.Text = "S&can..."
   $mnuScan.Add_Click({
     $lvItems.Items.Clear()
     
     [IO.Directory]::GetDirectories(
       (New-Object Regex("(?<=file:///)(.*)(?=GAC)", [Text.RegularExpressions.RegexOptions]::IgnoreCase)).Match(
         ([AppDomain]::CurrentDomain.GetAssemblies()[0].Evidence | ? {
           $_.Value -ne $null
         })
       ).Value.Replace('/', '\'), 'GAC*'
     ) | % {
       [IO.Directory]::GetFiles($_, '*.dll', [IO.SearchOption]::AllDirectories) | % {
         try {
           $an = [Reflection.AssemblyName]::GetAssemblyName($_)
           $raw = $an.ToString().Split(',');
           
           $lvi = $lvItems.Items.Add($raw[0], 0)
           $lvi.SubItems.Add((Get-Set $raw[1]))
           $lvi.SubItems.Add((Get-Set $raw[2]))
           $lvi.SubItems.Add((Get-Set $raw[3]))
           $lvi.SubItems.Add($an.ProcessorArchitecture.ToString())
           $lvi.SubItems.Add($_)
         }
         catch {}
       }
     }
     $lvItems.AutoResizeColumns([Windows.Forms.ColumnHeaderAutoResizeStyle]::ColumnContent)
     
     Get-ItemsNumber
   })
   #
   #
   $mnuExit.ShortcutKeys = [Windows.Forms.Keys]::Control, [Windows.Forms.Keys]::X
   $mnuExit.Text = "E&xit"
   $mnuExit.Add_Click({$frmMain.Close()})
   #
   #
   $mnuView.DropDownItems.AddRange(@($mnuFont))
   $mnuView.Text = "&View"
   #
   #
   $mnuFont.Text = "&Font..."
   $mnuFont.Add_Click({
     (New-Object Windows.Forms.FontDialog) | % {
       $_.MaxSize = 20
       $_.ShowEffects = $false
       
       if ($_.ShowDialog() -eq [Windows.Forms.DialogResult]::OK) {
         $lvItems.Font = $_.Font
       }
     }
   })
   #
   #
   $mnuHelp.DropDownItems.AddRange(@($mnuInfo))
   $mnuHelp.Text = "&Help"
   #
   #
   $mnuInfo.Text = "About..."
   $mnuInfo.Add_Click({
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
     $lblName.Font = New-Object Drawing.Font("Microsoft Sans Serif", 9, [Drawing.FontStyle]::Bold)
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
     
     [void]$frmInfo.ShowDialog()
   })
   #
   #
   $lvItems.AllowColumnReorder = $true
   $lvItems.Columns.AddRange(@($chCol_1, $chCol_2, $chCol_3, $chCol_4, $chCol_5, $chCol_6))
   $lvItems.Dock = [Windows.Forms.DockStyle]::Fill
   $lvItems.FullRowSelect = $true
   $lvItems.MultiSelect = $false
   $lvItems.SmallImageList = $imgList
   $lvItems.Sorting = [Windows.Forms.SortOrder]::Ascending
   $lvItems.View = [Windows.Forms.View]::Details
   #
   #
   $frmMain.ClientSize = New-Object Drawing.Size(570, 270)
   $frmMain.Controls.AddRange(@($lvItems, $sbStrip, $mnuMain))
   $frmMain.Icon = $ico
   $frmMain.MainMenuStrip = $mnuMain
   $frmMain.StartPosition = [Windows.Forms.FormStartPosition]::CenterScreen
   $frmMain.Text = "GACView"
   $frmMain.Add_Load({Get-ItemsNumber})
   
   [void]$frmMain.ShowDialog()
 }
 
 Export-ModuleMember -Alias gacview -Function Invoke-GACView
`

