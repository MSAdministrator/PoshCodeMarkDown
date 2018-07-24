---
Author: greg zakharov
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 4698
Published Date: 2013-12-13t15
Archived Date: 2013-12-16t12
---

# performance counters - 

## Description

screenshot at http

## Comments



## Usage



## TODO



## function

``

## Code

`#
 #
 $lvNames_Click= {
   $rtbInfo.Clear()
   $sbLabel.Text = "Ready"
   
   for ($i = 0; $i -lt $lvNames.Items.Count; $i++) {
     if ($lvNames.Items[$i].Selected) {
       $rtbInfo.SelectionFont = $bold
       $rtbInfo.AppendText("$($lvNames.Items[$i].Text)`n")
       $rtbInfo.SelectionFont = $norm
       
       $pcc = New-Object Diagnostics.PerformanceCounterCategory($lvNames.Items[$i].Text, ".")
       $rtbInfo.AppendText("$($pcc.CategoryHelp)`n`n$('=' * 55)`n`n")
       
       $rtbInfo.SelectionFont = $bold
       $rtbInfo.AppendText("Counters:`n")
       $rtbInfo.SelectionFont = $norm
       
       try {
       $pcc.GetCounters() | % {
         $rtbInfo.SelectionFont = $bold
         $rtbInfo.AppendText("   $($_.CounterName)`n")
         $rtbInfo.SelectionFont = $norm
         
         $rtbInfo.AppendText("   $($_.CounterHelp)`n")
         $rtbInfo.SelectionColor = [Drawing.Color]::Blue
         $rtbInfo.AppendText("   Type: $($_.CounterType), Lifetime: $($_.InstanceLifetime), ReadOnly: $($_.ReadOnly)`n`n")
         $rtbInfo.SelectionColor = [Drawing.Color]::Black
       }
       }
       catch {
         $sbLabel.Text = $_.Exception.Message
       }
     }
   }
 }
 
 $frmMain_Load= {
   [Diagnostics.PerformanceCounterCategory]::GetCategories(".") | % {
     $lvNames.Items.Add($_.CategoryName)
   }
   $sbLabel.Text = "Ready"
 }
 
 function frmMain_Show {
   Add-Type -Assembly System.Windows.Forms
   [Windows.Forms.Application]::EnableVisualStyles()
   
   $ico = [Drawing.Icon]::ExtractAssociatedIcon(($PSHome + '\powershell.exe'))
   
   $bold = New-Object Drawing.Font("Tahoma", 9, [Drawing.FontStyle]::Bold)
   $norm = New-Object Drawing.Font("Tahoma", 9, [Drawing.FontStyle]::Regular)
   
   $frmMain = New-Object Windows.Forms.Form
   $scSplit = New-Object Windows.Forms.SplitContainer
   $lvNames = New-Object Windows.Forms.ListView
   $rtbInfo = New-Object Windows.Forms.RichTextBox
   $sbStrip = New-Object Windows.Forms.StatusStrip
   $sbLabel = New-Object Windows.Forms.ToolStripStatusLabel
   #
   #
   $sbStrip.Items.AddRange(@($sbLabel))
   $sbLabel.AutoSize = $true
   #
   #
   $scSplit.Dock = "Fill"
   $scSplit.Panel1.Controls.Add($lvNames)
   $scSplit.Panel2.Controls.Add($rtbInfo)
   $scSplit.SplitterWidth = 1
   #
   #
   $lvNames.Dock = "Fill"
   $lvNames.FullRowSelect = $true
   $lvNames.MultiSelect = $false
   $lvNames.Sorting = "Ascending"
   $lvNames.TileSize = New-Object Drawing.Size(270, 17)
   $lvNames.View = "Tile"
   $lvNames.Add_Click($lvNames_Click)
   #
   #
   $rtbInfo.Dock = "Fill"
   $rtbInfo.Font = $norm
   $rtbInfo.ReadOnly = $true
   #
   #
   $frmMain.ClientSize = New-Object Drawing.Size(800, 470)
   $frmMain.Controls.AddRange(@($scSplit, $sbStrip))
   $frmMain.Icon = $ico
   $frmMain.StartPosition = "CenterScreen"
   $frmMain.Text = "Performance Counters"
   $frmMain.Add_Load($frmMain_Load)
   
   [void]$frmMain.ShowDialog()
 }
 
 frmMain_Show
`

