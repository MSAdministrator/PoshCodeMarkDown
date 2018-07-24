---
Author: janny
Publisher: 
Copyright: 
Email: 
Version: 2.39
Encoding: ascii
License: cc0
PoshCode ID: 4876
Published Date: 2014-02-04t13
Archived Date: 2014-02-07t23
---

# wmiexplorer - 

## Description

from greg�s repository on github.

## Comments



## Usage



## TODO



## script

`get-userstatus`

## Code

`#
 #
 set PSScriptRoot -val $(Split-Path $MyInvocation.MyCommand.Path) -opt Constant
 
 function Get-UserStatus {
   $script:usr = [Security.Principal.WindowsIdentity]::GetCurrent()
   return (New-Object Security.Principal.WindowsPrincipal $usr).IsInRole(
     [Security.Principal.WindowsBuiltInRole]::Administrator
   )
 }
 
 function Get-ImageFromString([Object]$img) {
   [Drawing.Image]::FromStream((New-Object IO.MemoryStream(
     ($$ = [Convert]::FromBase64String($img)), 0, $$.Length))
   )
 }
 
 function Get-NameSpaces([String]$root) {
   (New-Object Management.ManagementClass(
       $root, [Management.ManagementPath]'__NAMESPACE', $null
     )
   ).PSBase.GetInstances() | % {
     return (New-Object Windows.Forms.TreeNode).Nodes.Add($_.Name)
   }
 }
 
 function Get-SubNameSpaces([Windows.Forms.TreeNode[]]$nodes) {
   foreach ($nod in $nodes) {
     Get-NameSpaces ('root\' + $nod.FullPath) | % {$nod.Nodes.Add($_)}
   }
 }
 
 function Get-ClassesNumber {
   $sbLbl_1.Text = "Classes: " + $lvList1.Items.Count.ToString()
 }
 
 function Reset-AllMessages {
   $sbLbl_2, $sbLbl_3 | % {$_.Text = [String]::Empty}
   $rtbDesc.Text = [String]::Empty
   $lvList2.Items.Clear()
 }
 
 function frmMain_Show {
   if (!([AppDomain]::CurrentDomain.GetAssemblies() | ? {
     $_.FullName.Contains('System.Windows.Forms')
   })) {[void][Reflection.Assembly]::LoadWithPartialName('System.Windows.Forms')}
   [Windows.Forms.Application]::EnableVisualStyles()
   
   $ico = [Drawing.Icon]::ExtractAssociatedIcon(($PSHome + '\powershell.exe'))
   #
   #
   $bol1 = New-Object Drawing.Font("Tahoma", 9, [Drawing.FontStyle]::Bold)
   $bol2 = New-Object Drawing.Font("Tahoma", 8, [Drawing.FontStyle]::Bold)
   $norm = New-Object Drawing.Font("Tahoma", 9, [Drawing.FontStyle]::Regular)
   #
   #
   $img1 = "Qk1mAgAAAAAAADYAAAAoAAAADQAAAA4AAAABABgAAAAAADACAAAAAAAAAAAAAAAAAAAAAAAA//////" + `
           "//////////////////////////////////////////////AP///////////wAAAAAAAAAAAP///wAA" + `
           "AAAAAAAAAP///////////wD///////8AAAAAAAD///////////////////8AAAAAAAD///////8A//" + `
           "//////AAAAAAAA////////////////////AAAAAAAA////////AP///////wAAAAAAAP//////////" + `
           "/////////wAAAAAAAP///////wD///////8AAAAAAAD///////////////////8AAAAAAAD///////" + `
           "8A////////AAAAAAAA////////////////////AAAAAAAA////////AAAAAAAAAAAAAP//////////" + `
           "/////////////////wAAAAAAAAAAAAD///////8AAAAAAAD///////////////////8AAAAAAAD///" + `
           "////8A////////AAAAAAAA////////////////////AAAAAAAA////////AP///////wAAAAAAAP//" + `
           "/////////////////wAAAAAAAP///////wD///////8AAAAAAAD///////////////////8AAAAAAA" + `
           "D///////8A////////////AAAAAAAAAAAA////AAAAAAAAAAAA////////////AP//////////////" + `
           "/////////////////////////////////////wA="
   #
   #
   $img2 = "Qk02BQAAAAAAADYEAAAoAAAAEAAAABAAAAABAAgAAAAAAAABAAAAAAAAAAAAAAABAAAAAQAA////AN" + `
           "ju9gDYm1sA+O7jAMS3rQAUquEA/ez9ANrv9gDTZdIA2+/3AJeAbwCZMwAADGKBAI0tjAAOeJ4A/fD9" + `
           "AP/NmQD97f0A+q36ANxw2wAXmMgAbNbzAPuY+gCF4fUAUMvxAFDL8gA0wO8A997iAOm0fAC1YzUA+v" + `
           "TtABy17QDZbNgAa9f0AB217gA1wPAAHbXtALM8sgCF4PUAhuH0APnw5wD68uoAT8vxABy27gBr1vQA" + `
           "yXNDAIbh9QAvvu8ANMDwAPnx6QD///8AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA" + `
           "AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA" + `
           "AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA" + `
           "AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA" + `
           "AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA" + `
           "AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA" + `
           "AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA" + `
           "AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA" + `
           "AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA" + `
           "AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA" + `
           "AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA" + `
           "AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA" + `
           "AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA" + `
           "AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA" + `
           "AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAADIyMjIyMjIyMjIAAwAyMjIyMjIyMjIyMjIAMQsDADIyMj" + `
           "IyMjIyMjIyHh0CCwMAMjIyMjIyMjIECgIQHAILAwAyMjIyMjIyBDIDAhAcAgsDMjIyADIyMgQyMgMC" + `
           "EC0oADIyAAcAMjIEMjIyGwIpADIyAAcOBwAyBDIyBg0bADIyAAcOKw4HAAQyBiUTDREAMgEOKhovDA" + `
           "oKCiASFhMNEQAFFyEYMCIMATIGCBIWEw0RAQUuFRkjHwwBMgYIEggGAAAJBSYVGRokDAEADwgPADIy" + `
           "AAkFJywYFAEAMgAPADIyMjIACQUXFAEAMjIyADIyMjIyMgAJBQEAMjIyMjIyMjI="
   #
   #
   $img3 = "Qk02BQAAAAAAADYEAAAoAAAAEAAAABAAAAABAAgAAAAAAAABAAAAAAAAAAAAAAABAAAAAQAAxK+iAP" + `
           "///wD7/f8AVE5GAN+dfQBu6/8Aj6SsAP//9gBo7f8AVE9NAPzw6ADRyMEA0sjBAEphcABST0sAS2Fw" + `
           "AP///gACIS4AUk9MAHLh+QAVJzMAQaxTAFdNWQD+/v0Aeo+ZAOfr7QB6zeIA//nsADKy3wAVk8QAKZ" + `
           "c/ANnPyABhwd4AMDpPAP/l1gDCyNAA+OLSAKOsuABZeFsA+/TvAHx1cwCnkokAMp5BADG76gBdXGAA" + `
           "adv2AFOElQBXa4AAwsrOAH/j+QDf5OUA/d7LAP328gDRx8AAUmBnANq6qgC/yM0A/8WkAEBmUQCTtp" + `
           "kAhcyFAP/w4QCa06QAEajsAGrd9wD/6tIAGIwyAP77+ACAl6MAXnWEAAsQGwBccYAAHGYpAP/p3ABp" + `
           "nIkALqnWAPvu5gD/7eMAW21/AFjS8wCBprUAU1BMAIPh9gCknZYA/+LQACK6+gCw6/oAEAcKAP36+A" + `
           "BBPVAAT1ZlAEvH7QBYmK4AcMF9APHKtwD/6tUAcuL6AP7+/ABfoqYA//78ABRijgBRTksA+uneALCt" + `
           "rAD/+/kAQVxyANPJwgBWz/IAVL3cANTZ3QBOS0sAgJagAFCQjABAw+0AZ9f0AOXJuQBRcosAlbjEAO" + `
           "PZ0wBzhZMA4unpAIbT5QAdmcgAVlFNAB/Q/wD///8AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA" + `
           "AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA" + `
           "AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA" + `
           "AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA" + `
           "AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA" + `
           "AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA" + `
           "AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA" + `
           "AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA" + `
           "AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA" + `
           "AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAH19fX19fX19fX19fX19fX0obgkJCXtRDhIOZQN9fX19Hw" + `
           "ECAgcHBxs9X0EDfX19fQwBAABoAAAAAAAzA319fX0MAQICY1gnTGYkVAN9fX19agEAABAAAAAAACID" + `
           "fX19fTUBAgIBF0M0CgpJA319fX0LAQICAQFhZ2l2TQN9fX19CwF4RTIBMA1LRixTfX07SAReBnkPKQ" + `
           "0gFnwUVxFaJkIEOXMGUg8aNghZHHodZDoeBAQENwYxLgghLU9xKz9wKn19fX0ZLwhcBRNAa1tVYhV9" + `
           "fX19I1ZOBQUFYHJsdEo8fX19fSV1bVBEbxh3Rzg+XX19fX19fX19fX19fX19fX0="
   #
   #
   $img4 = "Qk32AgAAAAAAADYAAAAoAAAADgAAABAAAAABABgAAAAAAMACAAAAAAAAAAAAAAAAAAAAAAAA//////" + `
           "//////////////////////////////////////////////////AAD/////////////////////////" + `
           "//////////////////////////////8AAP///////////////////////wAAAAAAAP////////////" + `
           "///////////wAA////////////////////////AAAAAAAA////////////////////////AAD/////" + `
           "//////////////////////////////////////////////////8AAP///////////////////////w" + `
           "AAAAAAAP///////////////////////wAA////////////////////////aGhoAAAAsrKy////////" + `
           "////////////AAD////////////////////////Z2dkAAAAAAACnp6f///////////////8AAP////" + `
           "///////////////////////9nZ2U1NTQAAALKysv///////////wAA////////////////////////" + `
           "////////8PDwAAAAAAAA////////////AAD///////////9NTU0AAADHx8f////////Hx8cAAABNTU" + `
           "3///////////8AAP///////////9DQ0AAAAAAAAAAAAAAAAAAAAAAAANDQ0P///////////wAA////" + `
           "////////////2dnZfHx8AAAAAAAAfHx82dnZ////////////////AAD///////////////////////" + `
           "////////////////////////////////8AAP//////////////////////////////////////////" + `
           "/////////////wAA////////////////////////////////////////////////////////AAA="
   #
   #
   $frmMain = New-Object Windows.Forms.Form
   $mnuMain = New-Object Windows.Forms.MenuStrip
   $mnuFile = New-Object Windows.Forms.ToolStripMenuItem
   $mnuExit = New-Object Windows.Forms.ToolStripMenuItem
   $mnuView = New-Object Windows.Forms.ToolStripMenuItem
   $mnuTStr = New-Object Windows.Forms.ToolStripMenuItem
   $mnuSStr = New-Object Windows.Forms.ToolStripMenuItem
   $mnuPlug = New-Object Windows.Forms.ToolStripMenuItem
   $mnuHelp = New-Object Windows.Forms.ToolStripMenuItem
   $mnuInfo = New-Object Windows.Forms.ToolStripMenuItem
   $tsStrip = New-Object Windows.Forms.ToolStrip
   $tsLabel = New-Object Windows.Forms.ToolStripLabel
   $tsWMask = New-Object Windows.Forms.ToolStripTextBox
   $tsWLike = New-Object Windows.Forms.ToolStripButton
   $scSplt1 = New-Object Windows.Forms.SplitContainer
   $scSplt2 = New-Object Windows.Forms.SplitContainer
   $tvRoots = New-Object Windows.Forms.TreeView
   $lvList1 = New-Object Windows.Forms.ListView
   $lvList2 = New-Object Windows.Forms.ListView
   $chCol_1 = New-Object Windows.Forms.ColumnHeader
   $chCol_2 = New-Object Windows.Forms.ColumnHeader
   $chCol_3 = New-Object Windows.Forms.ColumnHeader
   $chCol_4 = New-Object Windows.Forms.ColumnHeader
   $chCol_5 = New-Object Windows.Forms.ColumnHeader
   $chCol_6 = New-Object Windows.Forms.ColumnHeader
   $chCol_7 = New-Object Windows.Forms.ColumnHeader
   $tabCtrl = New-Object Windows.Forms.TabControl
   $tpPage1 = New-Object Windows.Forms.TabPage
   $tpPage2 = New-Object Windows.Forms.TabPage
   $rtbDesc = New-Object Windows.Forms.RichTextBox
   $imgList = New-Object Windows.Forms.ImageList
   $sbStrip = New-Object Windows.Forms.StatusStrip
   $sbLbl_1 = New-Object Windows.Forms.ToolStripStatusLabel
   $sbLbl_2 = New-Object Windows.Forms.ToolStripStatusLabel
   $sbLbl_3 = New-Object Windows.Forms.ToolStripStatusLabel
   #
   #
   $mnuMain.Items.AddRange(@($mnuFile, $mnuView, $(
     if (Test-Path ($dir = Join-Path $PSScriptRoot 'Plugins')) {$mnuPlug} else {$mnuHelp}
   ), $mnuHelp))
   $tsStrip.Items.AddRange(@($tsLabel, $tsWMask, $tsWLike))
   $tsLabel.Text = "Filter:"
   $scSplt1, $scSplt2, $tabCtrl, $tvRoots, $lvList1, $lvList2, $rtbDesc | % {
     $_.Dock = [Windows.Forms.DockStyle]::Fill
   }
   $lvList1, $lvList2 | % {
     $_.FullRowSelect = $true
     $_.MultiSelect = $false
     $_.ShowItemToolTips = $true
     $_.Sorting = [Windows.Forms.SortOrder]::Ascending
   }
   $chCol_1, $chCol_2, $chCol_6, $chCol_7 | % {$_.Width = 130}
   $chCol_3, $chCol_4, $chCol_5 | % {$_.Width = 70}
   $chCol_1.Text = "Name"
   $chCol_2.Text = "Description"
   $chCol_3.Text = "Amended"
   $chCol_4.Text = "Local"
   $chCol_5.Text = "Overridable"
   $chCol_6.Text = "PropagatesToInstance"
   $chCol_7.Text = "PropagatesToSubclass"
   $tabCtrl.Controls.AddRange(@($tpPage1, $tpPage2))
   $tpPage1, $tpPage2 | % {$_.UseVisualStyleBackColor = $true}
   $scSplt1, $scSplt2 | % {$_.SplitterWidth = 1}
   $rtbDesc.ReadOnly = $true
   $img1, $img2, $img3 | % {$imgList.Images.Add((Get-ImageFromString $_))}
   $sbStrip.Items.AddRange(@($sbLbl_1, $sbLbl_2, $sbLbl_3))
   $sbLbl_1, $sbLbl_2, $sbLbl_3 | % {$_.AutoSize = $true}
   $sbLbl_2.ForeColor = [Drawing.Color]::DarkMagenta
   $sbLbl_3.ForeColor = [Drawing.Color]::DarkGreen
   #
   #
   $mnuFile.DropDownItems.AddRange(@($mnuExit))
   $mnuFile.Text = "&File"
   #
   #
   $mnuExit.ShortcutKeys = [Windows.Forms.Keys]::Control, [Windows.Forms.Keys]::X
   $mnuExit.Text = "E&xit"
   $mnuExit.Add_Click({$frmMain.Close()})
   #
   #
   $mnuView.DropDownItems.AddRange(@($mnuTStr, $mnuSStr))
   $mnuView.Text = "&View"
   #
   #
   $mnuTStr.Checked = $true
   $mnuTStr.ShortcutKeys = [Windows.Forms.Keys]::Control, [Windows.Forms.Keys]::F
   $mnuTStr.Text = "&Filter"
   $mnuTStr.Add_Click({
     $toggle =! $mnuTStr.Checked
     $mnuTStr.Checked = $toggle
     $tsStrip.Visible = $toggle
   })
   #
   #
   $mnuSStr.Checked = $true
   $mnuSStr.ShortcutKeys = [Windows.Forms.Keys]::Control, [Windows.Forms.Keys]::S
   $mnuSStr.Text = "&Status Bar"
   $mnuSStr.Add_Click({
     $toggle =! $mnuSStr.Checked
     $mnuSStr.Checked = $toggle
     $sbStrip.Visible = $toggle
   })
   #
   #
   if (Test-Path $dir) {
     if ((gci $dir).Length -ne $null) {
         if ([IO.Path]::GetExtension($_.FullName).Equals('.xml')) {
           $xml = [xml](cat $_.FullName)
           $plg = New-Object Windows.Forms.ToolStripMenuItem $xml.WmiExplorerPlugin.InputObject
           $dd += $plg
         }
       }
       $mnuPlug.DropDownItems.AddRange($dd)
     }
   }
   $mnuPlug.Text = "&Plugins"
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
   $tsWMask.Size = New-Object Drawing.Size(130, 23)
   $tsWMask.Add_TextChanged({
     if ($clone -ne $null -and [String]::IsNullOrEmpty($tsWMask.Text)) {
       Reset-AllMessages
       $lvList1.Items.Clear()
       $clone | % {$lvList1.Items.Add($_.Text, 1)}
       Get-ClassesNumber
   })
   #
   #
   $tsWLike.Image = (Get-ImageFromString $img4)
   $tsWLike.ToolTipText = "Set Filter"
   $tsWLike.Add_Click({
     if ($lvList1.Items.Count -ne 0 -and ![String]::IsNullOrEmpty($tsWMask.Text)) {
       Reset-AllMessages
       
       $lvList1.Items | % {
         if ($_.Text -notlike $tsWMask.Text) {
           $_.Remove()
         }
       }
       Get-ClassesNumber
   })
   #
   #
   $scSplt1.Orientation = [Windows.Forms.Orientation]::Horizontal
   $scSplt1.Panel1.Controls.Add($scSplt2)
   $scSplt1.Panel2.Controls.Add($tabCtrl)
   $scSplt1.SplitterDistance = 50
   #
   #
   $scSplt2.Panel1.Controls.Add($tvRoots)
   $scSplt2.Panel2.Controls.Add($lvList1)
   $scSplt2.Panel1MinSize = 17
   $scSplt2.SplitterDistance = 30
   #
   #
   $tpPage1.Controls.AddRange(@($rtbDesc))
   $tpPage1.Text = "Specification"
   #
   #
   $tpPage2.Controls.AddRange(@($lvList2))
   $tpPage2.Text = "Qualifiers"
   #
   #
   $tvRoots.ImageList = $imgList
   $tvRoots.Select()
   $tvRoots.Sorted = $true
   $tvRoots.Add_AfterExpand({Get-SubNameSpaces $_.Node.Nodes})
   $tvRoots.Add_AfterSelect({
     $lvList1.Items.Clear()
     Reset-AllMessages
     
     if ($tvRoots.SelectedNode) {
       $cur = 'root\' + $tvRoots.SelectedNode.FullPath
       
       (New-Object Management.ManagementClass($cur, $obj)
       ).PSBase.GetSubclasses($enm) | % {
         if ([String]::IsNullOrEmpty($tsWMask.Text)) {
           $lvList1.Items.Add($_.Name, 1)
         }
         else {
           if ($_.Name -like $tsWMask.Text) {$lvList1.Items.Add($_.Name, 1)}
         }
       }
       $clone = ([Windows.Forms.ListViewItem[]]($lvList1.Items)).Clone()
       
       $frmMain.Text = $cur + ' - WMI Explorer'
       Get-ClassesNumber
     }
   })
   #
   #
   $lvList1.LargeImageList = $imgList
   $lvList1.TileSize = New-Object Drawing.Size(270, 19)
   $lvList1.View = [Windows.Forms.View]::Tile
   $lvList1.Add_Click({
     Reset-AllMessages
     
     for ($i = 0; $i -lt $lvList1.Items.Count; $i++) {
       if ($lvList1.Items[$i].Selected) {
         $path = $cur + ':' + $lvList1.Items[$i].Text
         $frmMain.Text = $path + ' - WMI Explorer'
         
         $rtbDesc.SelectionFont = $bol1
         
         $wmi = (New-Object Management.ManagementClass($path, $obj)).PSBase
         $rtbDesc.AppendText("$(
           try {$wmi.Qualifiers.Item('Description').Value}
           catch {'Class has not description.'}
         
         $wmi.Methods | % {
           $rtbDesc.SelectionColor = [Drawing.Color]::DarkMagenta
           $rtbDesc.SelectionFont = $bol2
           $rtbDesc.AppendText("$($_.Name)`n")
           try {
             $rtbDesc.AppendText("$((' ' * 3) + $_.PSBase.Qualifiers['Description'].Value)`n`n")
           }
           catch {}
         
         $wmi.Properties | % {
           $rtbDesc.SelectionColor = [Drawing.Color]::DarkGreen
           $rtbDesc.SelectionFont = $bol2
           $rtbDesc.AppendText("$(
             $_.Name + ' (Type: ' + $_.Type + ', Local: ' + $_.IsLocal + ', Array: ' + $_.IsArray + ')'
           )`n")
           try {
             $rtbDesc.AppendText("$(
               $def = $_.PSBase.Qualifiers['Description'].Value
               if (![String]::IsNullOrEmpty($def)) {
                 (' ' * 3) + $def
               }
               else {(' ' * 3) + 'Not described.'}
           }
           catch {}
         
         if ($wmi.Derivation.Count -ne 0) {
           $rtbDesc.SelectionColor = [Drawing.Color]::DarkBlue
           $rtbDesc.SelectionFont = $bol2
           $rtbDesc.AppendText("Derivation:`n")
           $wmi.Derivation | % {
             $rtbDesc.AppendText("$($_)`n")
           }
         
         $wmi.Qualifiers | % {
           $itm = $lvList2.Items.Add($_.Name, 2)
           $itm.Subitems.Add($(if($_.Name -eq 'Description'){'See specification'}else{$_.Value.ToString()}))
           $itm.Subitems.Add($_.IsAmended.ToString())
           $itm.Subitems.Add($_.IsLocal.ToString())
           $itm.Subitems.Add($_.IsOverridable.ToString())
           $itm.Subitems.Add($_.PropagatesToInstance.ToString())
           $itm.Subitems.Add($_.PropagatesToSubclass.ToString())
     
     $sbLbl_2.Text = "Methods: " + $wmi.Methods.Count.ToString()
     $sbLbl_3.Text = "Properties: " + $wmi.Properties.Count.ToString()
   })
   #
   #
   $lvList2.Columns.AddRange(@($chCol_1, $chCol_2, $chCol_3, $chCol_4, $chCol_5, $chCol_6, $chCol_7))
   $lvList2.SmallImageList = $imgList
   $lvList2.View = [Windows.Forms.View]::Details
   #
   #
   $frmMain.ClientSize = New-Object Drawing.Size(800, 600)
   $frmMain.Controls.AddRange(@($scSplt1, $sbStrip, $tsStrip, $mnuMain))
   $frmMain.Icon = $ico
   $frmMain.MainMenuStrip = $mnuMain
   $frmMain.StartPosition = [Windows.Forms.FormStartPosition]::CenterScreen
   $frmMain.Text = "WMI Explorer"
   $frmMain.Add_Load({
     if (Get-UserStatus) {
       Get-NameSpaces 'root' | % {$tvRoots.Nodes.Add($_)}
       Get-SubNameSpaces $tvRoots.Nodes
       
       $obj = New-Object Management.ObjectGetOptions
       $enm = New-Object Management.EnumerationOptions
       $obj.UseAmendedQualifiers = $true
       $enm.EnumerateDeep = $true
       
       $sbLbl_1.Text = "Ready"
     }
     else {
       $sbLbl_1.Font = New-Object Drawing.Font("Microsoft Sans Serif", 8, [Drawing.FontStyle]::Bold)
       $sbLbl_1.ForeColor = [Drawing.Color]::Crimson
       $sbLbl_1.Text = ($usr.Name + ' is not an administrator.')
     }
   })
   
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
   $lblName.Text = "WMI Explorer v2.39"
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

