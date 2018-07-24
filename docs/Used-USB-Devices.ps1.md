---
Author: greg zakharov
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 4691
Published Date: 2013-12-11t16
Archived Date: 2013-12-13t12
---

# used usb devices - 

## Description

retrives usb devices from hklm

## Comments



## Usage



## TODO



## function

`get-currentusbdevice`

## Code

`#
 #
 function Get-CurrentUSBDevice {
   $dgvGrid.Columns.Clear()
   
   $dt = New-Object Data.DataTable
   $dr = $dt.NewRow()
   (gp $hash[$tsCombo.SelectedItem]).PSBase.Properties | % {
     if ($_.TypeNameOfValue -ne 'System.String[]') {
       $dc = New-Object Data.DataColumn
       $dc.ColumnName = $_.Name
       $dt.Columns.Add($dc)
       
       $dr.Item($_.Name) = $_.Value
     }
   }
   $dt.Rows.Add($dr)
   $dgvGrid.DataSource = $dt
 }
 
 function frmMain_Show {
   Add-Type -Assembly System.Windows.Forms
   [Windows.Forms.Application]::EnableVisualStyles()
   
   $ico = [Drawing.Icon]::ExtractAssociatedIcon(($PSHome + '\powershell.exe'))
   
   $frmMain = New-Object Windows.Forms.Form
   $tsStrip = New-Object Windows.Forms.ToolStrip
   $tsLabel = New-Object Windows.Forms.ToolStripLabel
   $tsCombo = New-Object Windows.Forms.ToolStripComboBox
   $dgvGrid = New-Object Windows.Forms.DataGridView
   $sbStrip = New-Object Windows.Forms.StatusStrip
   $sbLabel = New-Object Windows.Forms.ToolStripLabel
   #
   #
   $tsStrip.Items.AddRange(@($tsLabel, $tsCombo))
   $tsLabel.Text = "Registry Path:"
   #
   #
   gp HKLM:\SYSTEM\CurrentControlSet\Enum\USBSTOR\*\* | % {$hash = @{}}{
     $itm = $_.PSPath -split '\\';$itm = $itm[($itm.Count - 2)..($itm.Count - 1)] -join "\"
     [void]$tsCombo.Items.Add($itm)
     $hash[$itm] = $_.PSPath
   }
   $tsCombo.SelectedIndex = 0
   $tsCombo.Size = New-Object Drawing.Size(483, 23)
   $tsCombo.Add_SelectedIndexChanged({Get-CurrentUSBDevice})
   #
   #
   $dgvGrid.AutoSizeColumnsMode = "AllCells"
   $dgvGrid.Dock = "Fill"
   $dgvGrid.SelectionMode = "FullRowSelect"
   #
   #
   $sbStrip.Items.AddRange(@($sbLabel))
   $sbStrip.SizingGrip = $false
   #
   #
   $sbLabel.ForeColor = [Drawing.Color]::DarkBlue
   $sbLabel.Text = "mailto:grishanz@yandex.ru"
   #
   #
   $frmMain.ClientSize = New-Object Drawing.Size(570, 132)
   $frmMain.Controls.AddRange(@($dgvGrid, $sbStrip, $tsStrip))
   $frmMain.FormBorderStyle = "FixedSingle"
   $frmMain.Icon = $ico
   $frmMain.MainMenuStrip = $mnuMain
   $frmMain.StartPosition = "CenterScreen"
   $frmMain.Text = "USB Devices"
   $frmMain.Add_Load({Get-CurrentUSBDevice})
   
   [void]$frmMain.ShowDialog()
 }
 
 frmMain_Show
`

