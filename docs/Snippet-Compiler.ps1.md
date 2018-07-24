---
Author: greg zakharov
Publisher: 
Copyright: 
Email: 
Version: 3.5
Encoding: ascii
License: cc0
PoshCode ID: 4116
Published Date: 2016-04-18t10
Archived Date: 2016-03-25t09
---

# snippet compiler - 

## Description

now snippet compiler looks like this (see screenshot at http

## Comments



## Usage



## TODO



## function

`get-cursorpoint`

## Code

`#
 #
 $def = $(if ((gi .).FullName -eq (gi .).Root) {
            ([string](gi .).Root).TrimEnd("\")
          }
          else { (gi .).FullName }
        )
 
 ##################################################################################################
 
 function Get-CursorPoint {
   $x = $rtbEdit.SelectionStart - $rtbEdit.GetFirstCharIndexOfCurrentLine()
   $y = $rtbEdit.GetLineFromCharIndex($rtbEdit.SelectionStart) + 1
 
   return (New-Object Drawing.Point($x, $y))
 }
 
 function Get-Image([string]$img) {
   [Drawing.Image]::FromStream((New-Object IO.MemoryStream(($$ = `
               [Convert]::FromBase64String($img)), 0, $$.Length)))
 }
 
 function Invoke-Builder {
   $lstBugs.Items.Clear()
 
   if ($rtbEdit.Text -ne "") {
     switch ($tsCom_1.SelectedIndex) {
       "0" {$cscp = New-Object Microsoft.CSharp.CSharpCodeProvider; break}
       "1" {$cscp = New-Object Microsoft.CSharp.CSharpCodeProvider($dict); break}
     }
 
     switch ($tsCom_2.SelectedIndex) {
       "0" {$cdcp.GenerateExecutable = $true; break}
       "1" {$cdcp.GenerateExecutable = $false; break}
       "2" {
         $cdcp.GenerateExecutable = $true
         $cdcp.CompilerOptions = "/t:winexe"
         break
       }
     }
 
     $cdcp.IncludeDebugInformation = $chkIDbg.Checked
     $cdcp.GenerateInMemory = $chkIMem.Checked
 
     if ($lboRefs.Items.Count -ne 0) {
       for ($i = 0; $i -lt $lboRefs.Items.Count; $i++) {
         $cdcp.ReferencedAssemblies.Add($lboRefs.Items[$i].ToString())
       }
     }
 
     $cdcp.WarningLevel = 3
     $cdcp.OutputAssembly = $txtName.Text
 
     $script:make = $cscp.CompileAssemblyFromSource($cdcp, $rtbEdit.Text)
     $make.Errors | % {
       if ($_.Line -ne 0 -and $_.Column -ne 0) {
         $err = $_.Line.ToString() + '.' + ($_.Column - 1).ToString()
       }
       elseif ($_.Line -ne 0 -and $_.Column -eq 0) {
         $err = $_.Line.ToString() + ', 0'
       }
       elseif ($_.Line -eq 0 -and $_.Column -eq 0) {
         $err = '*'
       }
 
       if (!($_.IsWarning)) {
         $lstBugs.ForeColor = [Drawing.Color]::Crimson
         $itm = $lstBugs.Items.Add($err, 14)
       }
       else {
         $lstBugs.ForeColor = [Drawing.Color]::Gray
         $itm = $lstBugs.Items.Add($err, 15)
       }
 
       $itm.SubItems.Add($_.ErrorNumber)
       $itm.SubItems.Add($_.ErrorText)
     }
 }
 
 function Open-Document {
   Watch-UnsavedData
   (New-Object Windows.Forms.OpenFileDialog) | % {
     $_.FileName = "source"
     $_.InitialDirectory = $def
 
     if ($_.ShowDialog() -eq [Windows.Forms.DialogResult]::OK) {
       $sr = New-Object IO.StreamReader $_.FileName
       $rtbEdit.Text = $sr.ReadToEnd()
       $sr.Close()
 
       $tpBasic.Text = $_.FileName
       $tpBasic.ImageIndex = 2
       $script:uns = $false
     }
   }
 }
 
 function Save-Document {
   if ($rtbEdit.Text -ne "") {
     Save-WorkspaceData
   }
 }
 
 function Save-DocumentQuickly {
   if ($script:uns) {
     if ($src -ne $null) {
       Out-File $src -enc UTF8 -inp $rtbEdit.Text
       $tpBasic.ImageIndex = 2
       $script:uns = $false
     }
     else { Save-WorkspaceData }
   }
 }
 
 function Save-WorkspaceData {
   (New-Object Windows.Forms.SaveFileDialog) | % {
     $_.InitialDirectory = $def
 
     if ($_.ShowDialog() -eq [Windows.Forms.DialogResult]::OK) {
       $script:src = $_.FileName
       Out-File $src -enc UTF8 -inp $rtbEdit.Text
 
       $tpBasic.Text = $src
       $tpBasic.ImageIndex = 2
       $script:uns = $false
     }
   }
 }
 
 function Set-Opacity([object]$obj) {
   $ops.Checked = $false
   $frmMain.Opacity = [float]('.' + $($obj.Text)[0])
   $obj.Checked = $true
 }
 
 function Start-AfterBuilding {
   Invoke-Builder
   if ($script:make.Errors.Count -eq 0) {Invoke-Item $txtName.Text}
 }
 
 function Watch-UnsavedData {
   if ($script:uns) {
     $res = [Windows.Forms.MessageBox]::Show(
       "     Workspace has been modified.`nDo you want to save changes before?",
                  $frmMain.Text, [Windows.Forms.MessageBoxButtons]::YesNoCancel,
                                        [Windows.Forms.MessageBoxIcon]::Question
     )
 
     switch ($res) {
       'Yes'    { Save-WorkspaceData; $rtbEdit.Clear(); $tpBasic.Text = "Untitled"; break }
       'No'     { $rtbEdit.Clear(); $tpBasic.Text = "Untitled"; break }
       'Cancel' { return }
     }
   }
   else { $rtbEdit.Clear(); $tpBasic.Text = "Untitled" }
 }
 
 function Write-CursorPoint {
   $sbPnl_2.Text = 'Str: ' + (Get-CursorPoint).Y.ToString() + ', Col: ' + `
                                             (Get-CursorPoint).X.ToString()
 }
 
 ##################################################################################################
 
 $mnuITag_Click= {
   $tag = "//Author: " + [Security.Principal.WindowsIdentity]::GetCurrent().Name + "`n" + `
          "//Date:   " + (Get-Date -f 'HH:mm:ss') + "`n`n"
 
   if ($rtbEdit.Text -eq "") {
     $rtbEdit.Text = $tag
   }
   else {
     $rtbEdit.Text = $tag + $rtbEdit.Text
   }
 }
 
 $mnuFont_Click= {
   (New-Object Windows.Forms.FontDialog) | % {
     $_.Font = "Lucida Console"
     $_.MinSize = 10
     $_.MaxSize = 12
     $_.ShowEffects = $false
 
     if ($_.ShowDialog() -eq [Windows.Forms.DialogResult]::OK) {
       $rtbEdit.Font = $_.Font
     }
   }
 }
 
 $mnuOpaF_Click= {
   $frmMain.Opacity = 1
   $ops.Checked = $false
   $mnuOpaF.Checked = $true
   $ops = $mnuOpaF
 }
 
 $mnuWrap_Click= {
   $toggle =! $mnuWrap.Checked
   $mnuWrap.Checked = $toggle
   $rtbEdit.WordWrap = $toggle
 }
 
 $mnuPane_Click= {
   switch ($mnuPane.Checked) {
     $true  { $mnuPane.Checked = $false; $scSplit.Panel2Collapsed = $true; break }
     $false { $mnuPane.Checked = $true; $scSplit.Panel2Collapsed = $false; break }
   }
 }
 
 $mnuSBar_Click= {
   $toggle =! $mnuSBar.Checked
   $mnuSBar.Checked = $toggle
   $sbPanel.Visible = $toggle
 }
 
 $tsCom_1_SelectedIndexChanged= {
   switch ($tsCom_1.SelectedIndex) {
     "0" {$lboRefs.Items.Remove("`"System.Core.dll`""); break}
     "1" {$lboRefs.Items.Add("`"System.Core.dll`""); break}
   }
 }
 
 $tsCom_2_SelectedIndexChanged= {
   switch ($tsCom_2.SelectedIndex) {
     "0" {
       $txtName.Text = $def + '\app.exe'
       $chkIMem.Enabled = $false
       $mnuBnRA.Enabled = $true
       $tsBut11.Enabled = $true
       break
     }
     "1" {
       $txtName.Text = $def + '\lib.dll'
       $chkIMem.Enabled = $true
       $mnuBnRA.Enabled = $false
       $tsBut11.Enabled = $false
       $lboRefs.Items.Remove("`"System.Windows.Forms.dll`"")
       $lboRefs.Items.Remove("`"System.Drawing.dll`"")
       break
     }
     "2" {
       $txtName.Text = $def + '\app.exe'
       $chkIMem.Enabled = $false
       $mnuBnRA.Enabled = $true
       $tsBut11.Enabled = $true
       $lboRefs.Items.AddRange(@("`"System.Drawing.dll`"", "`"System.Windows.Forms.dll`""))
       break
     }
   }
 }
 
 $rtbEdit_TextChanged= {
   if ($rtbEdit.Text -ne "") {
     $tpBasic.ImageIndex = 1
     Write-CursorPoint
     $script:uns = $true
   }
   else {
     $tpBasic.Text = "Untitled"
     $tpBasic.ImageIndex = 0
     $script:uns = $false
     $script:src = $null
   }
 }
 
 $chkIMem_Click= {
   switch ($chkIMem.Checked) {
     $true  {
       $txtName.Text = [String]::Empty
       $lblName.Enabled = $false
       $txtName.Enabled = $false
       $chkIDbg.Checked = $false
       $chkIDbg.Enabled = $false
       $tsCom_2.Enabled = $false
     }
     $false {
       $txtName.Text = $def + '\lib.dll'
       $lblName.Enabled = $true
       $txtName.Enabled = $true
       $chkIDbg.Checked = $true
       $chkIDbg.Enabled = $true
       $tsCom_2.Enabled = $true
     }
   }
 }
 
 $mnuICnM_Click= {
   $script:buf = $lboRefs.SelectedItem
   $lboRefs.Items.Remove($lboRefs.SelectedItem)
 }
 
 $mnuIIns_Click= {
   (New-Object Windows.Forms.OpenFileDialog) | % {
     $_.Filter = "PE File (*.dll)|*.dll"
     $_.InitialDirectory = [Runtime.InteropServices.RuntimeEnvironment]::GetRuntimeDirectory()
 
     if ($_.ShowDialog() -eq [Windows.Forms.DialogResult]::OK) {
       $lboRefs.Items.Add('"' + (Split-Path -leaf $_.FileName) + '"')
     }
   }
 }
 
 $frmMain_Load= {
   $rtbEdit.Select()
   $txtName.Text = $def + '\app.exe'
   $sbPnl_2.Text = "Str: 1, Col: 0"
   $lboRefs.Items.Add("`"System.dll`"")
 }
 
 ##################################################################################################
 #
 #
 $i_1 = "iVBORw0KGgoAAAANSUhEUgAAABAAAAAQCAIAAACQkWg2AAAAAXNSR0IArs4c6QAAAARnQU1BAACxjwv8YQUAA" + `
        "AAgY0hSTQAAeiYAAICEAAD6AAAAgOgAAHUwAADqYAAAOpgAABdwnLpRPAAAAQRJREFUOE+Vkr0OQ1AYhk8ns7" + `
        "gR9+MnEbGJhQswGSsxGCxCJHSSGI1GsbF1dAUmMbWvnv6qavskQo73Od/5fHZRFA3D0Pf9OI7TNJE1GIZhWZb" + `
        "neY7jiOu6xwunTZqmcRwnyzJimibSSZLQPFZXRQhA13WCC0IYhtsCAm3bKopCNE37UaiqakX41Ijv+2VZSpI0" + `
        "V+i6DhVxesr+xn0FDxCKongR4CxA6JmHgG7e01hZCHmei6I4Hwnf6+v2kNM0fRG2B4e+4ji+CnVd0zkcPoBXE" + `
        "JBZCmhmFSoEQfBHBfplBUEghmFghGC7B6Q9z1NVleBm2zb+KIwdYDQUVH8Ge8uybFnWGWKTD130GLf2AAAAAE" + `
        "lFTkSuQmCC"
 
 $i_2 = "iVBORw0KGgoAAAANSUhEUgAAABAAAAAQCAIAAACQkWg2AAAAAXNSR0IArs4c6QAAAARnQU1BAACxjwv8YQUAA" + `
        "AAgY0hSTQAAeiYAAICEAAD6AAAAgOgAAHUwAADqYAAAOpgAABdwnLpRPAAAAQ9JREFUOE9jvFlZyfbqFdOZMy" + `
        "xv3rB8+sSADfzh4/spL/85MfG9qirDw8jI/8ePgxB+sHbtR0fH683NDI9VVUGqk5Mh6oGi2DWuXft/7dqn0tI" + `
        "MQEyUhuTkP7t2PWRlZXjJy0ukhl+zZmHRgNMjwcF/+/thGvbvB/oB6HoIeu/tDUFwESDje3Dwl7Y2FA0gf6Oh" + `
        "4GCgOjhCaAD6BotqoGZUDd+rq69xcIA9DQwyDOORzYaw3xcXo2rAG3E/PT2RNCxdCom4587OWBFQChgGz7OzY" + `
        "TbANLzcvBkrwqkBjw3AkL2fnHyGiwuUloBRCET40x5QNRBdERJiAFK3XV2BKQoYKcgI6FxkBDQbqPq0qSkAg2" + `
        "nz+rHom20AAAAASUVORK5CYII="
 
 $i_3 = "iVBORw0KGgoAAAANSUhEUgAAABAAAAAQCAIAAACQkWg2AAAAAXNSR0IArs4c6QAAAARnQU1BAACxjwv8YQUAA" + `
        "AAgY0hSTQAAeiYAAICEAAD6AAAAgOgAAHUwAADqYAAAOpgAABdwnLpRPAAAAQhJREFUOE+Nki9TxDAQxRfHSQ" + `
        "bDDJ/gZGUt8nB8hcjIyNiTkZGxkZGVsSsri6utDK43GO7lWsr9CYXM67Szfb/dzSZ3xnNK9D6MHwcaD1Rcm3t" + `
        "6eqDX7eb58ZOU5bYfoa/VFdskTGsD005HuHUYJj+iRRAA9KIi4cmA/wOAgbtUyYZq+V8g8FAAftuIcr2PQyVO" + `
        "FbjLLaH7SdLOWiL4UK5zTX8BgLkSssK36BxIt25ErgCbK4TcEuZVSv+TeypiQncBrB+ctJ3x30BzqgDAsCgKv" + `
        "zCJvW/nCgvAfSxqBtwNsFIBk9WOK+FppyKOEFrfA9zaci0D4SX2ETcKx54lFgV0fCZfi/CmwxH35x1B2WVBxw" + `
        "AAAABJRU5ErkJggg=="
 
 $i_4 = "iVBORw0KGgoAAAANSUhEUgAAAA4AAAAPCAIAAABbdmkjAAAAAXNSR0IArs4c6QAAAARnQU1BAACxjwv8YQUAA" + `
        "AAgY0hSTQAAeiYAAICEAAD6AAAAgOgAAHUwAADqYAAAOpgAABdwnLpRPAAAAZhJREFUKFN9kN8rQ2EYxx8u3L" + `
        "p244Yb/wB/ADcKV25FaCEs5kcyJLtxs7TFBeXC2hDpkEhIETXLafmxzTBtMsx2Zud4OUMez/bmtKi9fXou3vN" + `
        "5v8+3kyO79XmFxQDwsGksqDRBblHy3jd1XkM3/EQ9ntaOKkQA9WXjebVMlayyq5qma7FElSyMsURCjsRe3P4n" + `
        "w8iaY/fOH/kG+pyC+elNakoW4hvx7QOVJJ4GGam18zHb5g3c2vNFZwV5yU9GU3SWP6+Wci/CUAy8ksr5TaXIj" + `
        "FTuhRUMSCh6URCwe0hIdU31ixrSXS3H6a6adxXDs0cUw9g1uAKJC3O6n5X2pntb6Ibnad7RHXYOLMNJ6IswzR" + `
        "wnWDJzb6a3H8T2vgVwBb+I0cmDuKL+30t55G3fYJvBDofXH8SweSccf/+/l3vrl9iin4M9n0oYxzdCUfann+Y" + `
        "JXtS1z8LW2RsxMCZcPbz+6cfzyFs8x+bWaVgTFaJ/ZMl7r/D/ovXTPNspNuqmYNkpE72Djn0Rs1DfZAXHYZzo" + `
        "6bdlp65h4gfkvOeqbYKaEwAAAABJRU5ErkJggg=="
 
 $i_5 = "iVBORw0KGgoAAAANSUhEUgAAABAAAAAPCAIAAABiEdh4AAAAAXNSR0IArs4c6QAAAARnQU1BAACxjwv8YQUAA" + `
        "AAgY0hSTQAAeiYAAICEAAD6AAAAgOgAAHUwAADqYAAAOpgAABdwnLpRPAAAAjxJREFUKFNjvP36HwMO8O/Lve" + `
        "UbjzD8YGAQELDWllDQMAMpvPnyHxq68eIvRCShYsKOA6eBjPfvXjVMWHD28jWgCMPuU3fXLOlHRkCR6y/+oaE" + `
        "z564FZEwAIgag0luXFnz/sBqIfr5b9vv1/FULe3eevHP1+V9kBFEN0rB6Ud+3d8vhqv++mPH7SuHyeZ2LZ7bN" + `
        "n9I0a0Ld9J6qyR3ll5/+hSCGFQu6v71aADH734sZ/29V/b+SjoaAGi4+/QtBDEvndHx+NguieueabiDavqpr8" + `
        "7KO9YvbVs1vXjarceHU+rkTa2b2VU3trgTqZFg8q+3Tw0lAl9w70rBhed/fe43/H7Qg0P3G/3er/98o+X8t++" + `
        "LKsCWz+xnmT23+eKfz/6OejSt6j+/qAcnBnXQh/v/ZkD9Hvb/vdry9SK8kPXDehqMMcyY2vL/e+OBo7YZlPf/" + `
        "uNvy/nAHScDHp/9mIP8f9vu9z+7TN+s0aw8ZM1YVTq47e+cUwo6/2zaXKtUu6zx3q/H+j8P/l1P/nYv6dDPx1" + `
        "0PPrTvt3G0xertA82yoQ7a2UlFe3/8YPhqldVZf31a1b0vX/Tt3/8/H/T4f+OuLzdY/jh83mr1fpPV2gcqVHu" + `
        "CFdOTczbuflb0DEMKm9fPXCziubss72M0DQ8R6WfR0cW5r4V9aIzi+XnlYkH+OtFJ1Zs+ncZyBi6G8p7W9Ia6" + `
        "9MqM4Lz0/ySwxzCfKwdLbSM9VT1VSSlpMUERPikxDmW3PyEwQxLDv6niQEAAw6z50R4Nz2AAAAAElFTkSuQmCC"
 
 $i_6 = "iVBORw0KGgoAAAANSUhEUgAAABAAAAAPCAIAAABiEdh4AAAAAXNSR0IArs4c6QAAAARnQU1BAACxjwv8YQUAA" + `
        "AAgY0hSTQAAeiYAAICEAAD6AAAAgOgAAHUwAADqYAAAOpgAABdwnLpRPAAAASxJREFUKFOVj8tKw0AUho/PVO" + `
        "gDdNFdkbxE3yB9BEHovgSyFgOC1JRKV5OLYWq8BClao91URqx00aAo0mjwDwNxKs0ih7Oa833nn7PzuMyoUsW" + `
        "vmb43arWMbpedlRRGAKbiAzBNF7nQbl/idV1SGAHInUVGty8/qsAYcxzH8zzf95EXBAGWFAJgmjxvCFszCgEw" + `
        "3WwKrutiNwq7UZxzNQEwRaJCAmC6evqWN8i7t7ac4mjAdDH/E+SgrCEApmC2LhLwNB7Per2w3w8ty7JtO4rua" + `
        "jWjXjdlAmBi91+qwPm80TjsdHI6juM0TSGgpQCYRpNPVRgOI007xSWSTpJEFQDTyfW7Kpjmsa57g8GDpIUQqg" + `
        "CYjs7f7HC5q+0jsdk08JN/JQVM0YDpIFhV6l80oM4zNkn6agAAAABJRU5ErkJggg=="
 
 $i_7 = "iVBORw0KGgoAAAANSUhEUgAAABAAAAAQCAIAAACQkWg2AAAAAXNSR0IArs4c6QAAAARnQU1BAACxjwv8YQUAA" + `
        "AAgY0hSTQAAeiYAAICEAAD6AAAAgOgAAHUwAADqYAAAOpgAABdwnLpRPAAAARZJREFUOE9jvP36HwMMzJq9+8" + `
        "bxB/b2ahYWrHBBOOPEid8HD95iuPnyHxyVtu1MSzvT07P/FzYAFAfKMlx/8Q+IgEp9fWcChY5gA0BxoOzMmYd" + `
        "AGq4+/wtExJgN1APSQLzZCA0E3Q1RikXD/v37Dxw4cOjQIaBHjh49CvQ5xKM4NWCGDQENBw8ePAwGQOOPHz9O" + `
        "VRsg/sZEQHHsfoBI4EJwTwMZoHiAG3PixJ3p009t3Hhq8+bN589fMzaeaWY2B9mGTec+o2g4fvyhk9OKsrKVN" + `
        "2/e/P37N1ADEEE0QNCak59QNMyZs7609NCWLbc+fPjw5MkTZA3Ljr6HIIaVR557ejYBdbu5zQS6BBlANEDMXn" + `
        "DwLQQBAA5O4fpSVOxBAAAAAElFTkSuQmCC"
 
 $i_8 = "iVBORw0KGgoAAAANSUhEUgAAAA8AAAAOCAIAAAB/6NG4AAAAAXNSR0IArs4c6QAAAARnQU1BAACxjwv8YQUAA" + `
        "AAgY0hSTQAAeiYAAICEAAD6AAAAgOgAAHUwAADqYAAAOpgAABdwnLpRPAAAAcRJREFUKFNjXLzr0dWHn0IcFX" + `
        "h5uRjAgJGB4dKN54Ur7z/+z83AxMTw9m2aCW9hlD4TMwuDTdWZhJ3/C2Zcuf7iHxBde/rLPG8/Q+1jhqbnDFU" + `
        "PGIpvMuRcYkg+KRO87uaLfwwqRWdMV//Pmn3j6vO/By9/kMk4zNDykqHuMUPo7qo5l+duuScTuQWoGoh6Vt5k" + `
        "4Mk6wzXnf+rMGwcufWAI2MbQ8JQh95J/3dFLj39efvoXiIAMBt/NDGlnZGK2MzCknmaY8E0mfjdIac1Dhszzl" + `
        "bMuXn729+JTBErpPwdUDdLDEH+UofU1iFN2B0h2rbh5/slfNISkOmo/Q9V9kG9ST8v4rli27wkEnX/86+yjP0" + `
        "B0/vEfBs/1MJcE72TIv8qQfhZkPBwln0zuOnX64R8gWrr3CUPCMaAvO5bfZGBwX8sQuR8dhe1pW3rj6J1fx+9" + `
        "8hxgMdPSJuz8Z9t/4gQsduvnDJG0HSGnCsZbF14HKGHZe/oYV7bn63at4Dyik084Yp+4AcoHKGDad+4wVeRbt" + `
        "YYg5BFQqFb1t+4WPEDUMa05+wopyJ5wFqXZbs/zAS7gChmVH32NFq098iGo4svLwK2RZAH0Sgt0BWzw0AAAAA" + `
        "ElFTkSuQmCC"
 
 $i_9 = "iVBORw0KGgoAAAANSUhEUgAAAA8AAAAOCAIAAAB/6NG4AAAAAXNSR0IArs4c6QAAAARnQU1BAACxjwv8YQUAA" + `
        "AAgY0hSTQAAeiYAAICEAAD6AAAAgOgAAHUwAADqYAAAOpgAABdwnLpRPAAAAbRJREFUKFN1kE8oQwEcx3+Ui4" + `
        "OLOOCipNyJlCs5LMrFDCn/DoqlTIqkXNiBHGTcdljkT1gbCz02Yxtmsz+2mv/mz9Cebe2fvT2/19teQ16fw/d" + `
        "93+f7/qU5X+MAEKdiS9tO6wd98xauKMxsrCrIzcnCnjs8b75gKAqXL3G1lSzpJlq36cpVOk9Kp8/TMHxf3kvY" + `
        "3VG8imCAeiU0q9GmSvs01Rt00cx7kdBQLjoGoR0mvDD2BDz5gYW0PVOm2xDwCeg6BYXOUyzxZUsiQwsXtifK4" + `
        "qaMN6G6UR10n0GPGRpU+xckNgm7Y8qYsUBDv8P8EDG7KQ7xohNV5n4NKrHMnrTnLmEmDB168+PX+SOVyiRKyQ" + `
        "Hz0riUbF7D+Cv0WndN3rP7WILbiMEVRAZnjYkBqojOFYQWDdplPXsndzGEaXhy5tEsbUeMx6J1Rcs6FUziE9K" + `
        "dBzw9dAR+2NwMA+EIEzY/8ztxwJNjZpp/AJUliEyvXbHfkS9Q7lkDbPkX2DT6EaUpUDtAQLseB3kC5Zbpk+1/" + `
        "ASt6H8v6ib9GpMYB2vJTkutTA8i0Xo5lHdk0ol7SeFLL1PwNGc+R5HRDfbAAAAAASUVORK5CYII="
 
 $i10 = "iVBORw0KGgoAAAANSUhEUgAAAA4AAAAQCAIAAACp9tltAAAAAXNSR0IArs4c6QAAAARnQU1BAACxjwv8YQUAA" + `
        "AAgY0hSTQAAeiYAAICEAAD6AAAAgOgAAHUwAADqYAAAOpgAABdwnLpRPAAAAXhJREFUKFNtkGtLwlAch2d9uj" + `
        "5J0Msu2G1oF5PMoCAQAstMtJUmrSIsKBR8oYmXOW9QiKbNSHGmw6nTf2duzpEdHthhPPx4OJqDe6bd4jDVqab" + `
        "Ts3Mzmqlp9U/xvk9+AIAgCDzPcxxXZxsLOtLyXE6X+NzXQA1m9rx3BSSLRxhAgwekSmQqAzXYrivb7Iy9Whsy" + `
        "THfvpo7UFNNXg+3YqW8OJMzXeSPx9gdxnukly31s2xopsuCkRExErtuXY9CnIwA77DG486nPHrZhCeaqIGF00" + `
        "K1hjOIxzXE6pjt8SVTgghYxnERRhtorsEAV4DEA8+teDDf7wiWQ2DoOFRswiayuGe+CRXAlRfRHAZQhdauR1Z" + `
        "VN72sWJJCKYiaRVS1+ubTqlMBND+EyXKWAQOlJcdhBwVl81EpGmwqrBhLFTCKvukM/Cst6j9SNxuwJsMXhNAb" + `
        "W6GjVGawraHEiQIvRoQygi5+Gpzj4YnAbGT6WzV9TQLmL2vN/QeovzxcByOFi9/gAAAAASUVORK5CYII="
 
 $i11 = "iVBORw0KGgoAAAANSUhEUgAAABAAAAAQCAIAAACQkWg2AAAAAXNSR0IArs4c6QAAAARnQU1BAACxjwv8YQUAA" + `
        "AAgY0hSTQAAeiYAAICEAAD6AAAAgOgAAHUwAADqYAAAOpgAABdwnLpRPAAAAahJREFUOE+VkdtKAkEYgKc36Q" + `
        "V6hx6l2wgi6qKIgkpCkCiIDmwHEUvR6EBFhRTZubTcdHW3RE07bEes2Mx2t82/f3ZEZLtq+C5mdr9vZtitSb2" + `
        "UyL9G8qlUjW3QPcTNMcKibHmLS3LxWEIyJ823sQYn1xYIhgUpwxgbtQcn6wRvbVaWmYYQ8eEnuVoPP8+ZXKx3" + `
        "wGVhyWfXlMgZRzUGictmYFwBCMo9Z0FT1vE5BqgxSIwF31GAAKJo+wOLWbs/XU1r1xJqDHJ+Zwb6DoCL4fAnA" + `
        "cAwDF3XVVV9LWgYbEXzaCIkcmPQQF0G4Bj22URBx4QOnLx+AQYMlMnptRl8zYA6zOh38rIC1ZxJ4PHRDGVymN" + `
        "ZpUByHdwfDNhFCe/kSFi9gQYI5EfwJOBJogDIJXqo0+ByCxz5Gz8ie5QRcHks0QJkE4sVykOsEExawjb0CeAR" + `
        "wR+FApAHKZIX/KAdSC5h0Ozb+nrBtXgllMh9S1vh3/DXKVZOSakQ6+1f+BpsxGqBMvIdvSCByiw2jo2ehchNX" + `
        "FKZ5mOJhnacBmsS1m7fQ3u2vfHjLBM1f7Ns42ve2ZLkAAAAASUVORK5CYII="
 
 $i12 = "iVBORw0KGgoAAAANSUhEUgAAAAsAAAAPCAIAAAC9X6JnAAAAAXNSR0IArs4c6QAAAARnQU1BAACxjwv8YQUAA" + `
        "AAgY0hSTQAAeiYAAICEAAD6AAAAgOgAAHUwAADqYAAAOpgAABdwnLpRPAAAAX9JREFUKFN9i0tLAnEUR2+bIC" + `
        "hatK2PEhQRJkK0CmlRu1ZBSC9aSVkhQWAFmpqZPZymDJ3UStORKCnthZjmlCWWQ0aEkVhhM9UdBqRVw+Fy/uf" + `
        "HlDFP38WPPACUV1ThLX0/3Eeh8F5ZXQNX2e94pqBQ6pgsj16if9SAHZ8Qe8SB6+5T+8NJdBH0rh4VdnSIsjxC" + `
        "uo+0Znuc/ULHu7LpM5E+cYIIyyPHiedNp89IeNFnLe6VdTcWcYKLDC8yqVvTmQgjSc8YbePTqxGWEzuc3XMin" + `
        "uOk3kINqebGNcvopQ4naa6EYsTg2DkYHDP9jRBMFhE6+ra091Ar0XcOLNS1GLSOWyziBIHE56zjDlpd0O4DGR" + `
        "WI5UFqBzkNbdvYcQU6XoAmcn47PWq+BLnfE32HRkJjuxGeUruwus9y0GDdOs9PkddSZQhF2uvVu1K7kTx2YXW" + `
        "evkLzhmT4EH/VUilb6A0v1C8LRUYJqzWYUxNMx0RYbU2gi6ALhWDQYXH/5X9+AaM1WpeblNK7AAAAAElFTkSu" + `
        "QmCC"
 
 $i13 = "iVBORw0KGgoAAAANSUhEUgAAABEAAAAPCAIAAACN07NGAAAAAXNSR0IArs4c6QAAAARnQU1BAACxjwv8YQUAA" + `
        "AAgY0hSTQAAeiYAAICEAAD6AAAAgOgAAHUwAADqYAAAOpgAABdwnLpRPAAAAZNJREFUOE+VkNlOwkAUhsfn8Q" + `
        "F4Gx/C5c5o3KiBqFEjCFHiCu4IGCIhaqopLqAoBGVpUISqoEjYsZ1pPWMTLrxowuTLn5kz/5k/Z7oSeRl1up4" + `
        "+ZEDn1KmbNhoVFHsnnYKiAgHg1QHG6rnk1aN2BT0IBICefsYKqu41Kq5LHoVz1KTSx1gPA7x2BW7R7SsGoKFX" + `
        "bzngUupRu4KCGTzrLTp8Hzv+wro7u3n0tubKWBwx80bEshoy265NSwFg3sQaZ73G6WPwI44XbQGlvbCi1LHyW" + `
        "VeEMnkpkmQBfhXfC9JNBnNpyTDlBT86i7fMp03o6e65qIug7FdNBs2WCGg8j0FDWarnvMgYPOBHvmhjxpOHnp" + `
        "Yk137kUkMuVGWhLD9/QwimITkczEhcGrO8NM44wY/cd9XJ3WdRom9XmlTfK1TTXzQn8kYTrl4k0JOkOKbfBz/" + `
        "au64w23wTQlp/ITVCJ/kmKToJCeewOgmbkvzxn6GRbfAjO1eCnuX9BGDdelRZtEf//dvcwtnohHNw2AF+tMIW" + `
        "O+UX9FIdqJLevT8AAAAASUVORK5CYII="
 
 $i14 = "iVBORw0KGgoAAAANSUhEUgAAABEAAAAQCAIAAAB/UwMIAAAAAXNSR0IArs4c6QAAAARnQU1BAACxjwv8YQUAA" + `
        "AAgY0hSTQAAeiYAAICEAAD6AAAAgOgAAHUwAADqYAAAOpgAABdwnLpRPAAAAbhJREFUOE+VjWtLInEYxf99nj" + `
        "5A36aX3YiKdoPYV1GWBEEUkTpQtBBdyO5EVHQZzDXJrkOlNlZWZppmXrP5X6YzDAzRLmw9HH4cDufwVFw+CfL" + `
        "dCyUEVDVfZZqveHLxKCxh+V+PAjmLcwht03zFEyXGPwrjNru07FNhzNxKrBo5iXHHZgk0hcZPu7ToU/+Z/HL+" + `
        "QU6O7rkkMxBtkz/skuU/Jc19K0jIwS1zbL2CEKqmWntcc97Lv5MG2yxCsh9lzh0N1HUdxKCl25XIcXj1iVnJb" + `
        "oTC19tmQOJVtaH1LAYwJU2AqSIHb9IGlQeDvmsKriqFBpsbhmwHy/3LCWwqq+XCmwCTeQ5GUgZPYgz0XlFw5T" + `
        "hf1zmNPllTSr0z10zor1TkyiJdFPGsiD7zcJKdxdnhHdu7oZ4I3QzThYNMTcck+mTpMG+fVjVm/MmWjD+xF4P" + `
        "BpPEhcGtQVvFnx+1P13ZOoU/c/hw2ZaoXyuK5KBJ5fpfhaoqfx9nxPfNHqeeKbofp2oU2H8g1Ng2gTya8GWxG" + `
        "ZkOQNHVuyjmhOMZPXb8DwyN7Q5Jn0Cn3D250dLnb2kfRJ2Ny+rt6BzW7RZLF6CiHAAAAAElFTkSuQmCC"
 
 $i15 = "iVBORw0KGgoAAAANSUhEUgAAABAAAAAQCAIAAACQkWg2AAAAAXNSR0IArs4c6QAAAARnQU1BAACxjwv8YQUAA" + `
        "AAgY0hSTQAAeiYAAICEAAD6AAAAgOgAAHUwAADqYAAAOpgAABdwnLpRPAAAAsZJREFUOE9FkmtIk1EYx9/nvH" + `
        "u3V+dUyltIIaI5DAsiyy5SlphmCprpirnp8MWaoEikXwzayrQgzTLtQ0hN0UIssyQv1AcnQnMaXlIzTDOsvG7" + `
        "OnG7qTmdq+vBwLpzf/znP4fwBY0z9D6O+k6qpdlha5lOwsri4zDI2eSqz/4DQyWmLoYiAhNlg+JsQjzkOFxXh" + `
        "ykpcUYELCnBGBo6Onj1xfH5iYgOzV9+gLafDcGoqVqlwYSHOzcUKBU5JsWd8PA4NnQvY211fty2Yi47CERE2i" + `
        "eRDerotONjm7W1xdV11d1/192+NicFBQSaxWO/jM/Fz3H7DtE5nDgzEISHNKtXIzExTVtYUwC+A3wBPpFL96K" + `
        "g2O9vg5TXg6VkadtIuGEpKMPj62vz8Rjhu1mIxLi015+aOATy7cvWP0UiIaaXyh0jU6eKicXXt7+ulevcFjnl" + `
        "4kBqTLNsnkxms1sWVlX6t1mQ2E3qc477weH1CYZujY4ODw7VMJdW+c0ePs/N3F5dhgaAfoFuWvGyzEXQN428K" + `
        "hQ5AzzA6gaCVz28QCKRxsVSrl6eWZT+zbA+f3wVQl5NjXSMwJqI2tbodoJ2mP/J4r2i6imEuno+img4famSYN" + `
        "ob5hFCJXD41P0/oSY1mBeMFq/VldnYLQANCGoTu03RSQhzVVVJcw+O9BXisVE6uv7KP494BdHCcaXXVZLHU5+" + `
        "VVkVOaTqbp/PyblMlorBYKNQCNEgmp2p2W9hrgDUJkbJHLib41MfEBwD2EgtzchgYH7D/dUV5WjlAZQK1Y/By" + `
        "gmiRClQBPAaoCAu4C5CMUSdPy5Eub1iDTe7W6mHQJUAJQShoAeAhAtvkAKoBYhI4dPTK/3vCm+ciq/lHJDZHo" + `
        "DkK3AW6tcyQzETrIspFnw41Gw7aXtpw4/HXwuuyyNFB8YZfXOU+PkD27w8+cqq19sbBg2mL+AY/qjcgixtNMA" + `
        "AAAAElFTkSuQmCC"
 
 $i16 = "iVBORw0KGgoAAAANSUhEUgAAABEAAAAQCAIAAAB/UwMIAAAABGdBTUEAALGPC/xhBQAAAiVJREFUOE+Fkl1oU" + `
        "nEYxn3POR71THexsRzdNeoiggiSYIOii1XU7vqgLptRV8bqJtwownBhJMJgFH3c72aJsaSty7wpL4yam1qp6d" + `
        "xRdzx6znHnWPnx5tlcK1305+HPw/Py4+GFV4MdT5YlUZisVLjO0Wai6RwUCrfqNWO5bJiZebUj1s4oCs/lANH" + `
        "SqJlcrqFIJNGJtTN51i4LNCKD9X6Bo2226f8wilIspIm1bPdNGzk/b8Yqfc9xLJVabcP+6smnxxVO63AwBth3" + `
        "YMCC9d7CN73b7f0no8iltQiF301PH9Em4sjI4CHEnoZI37EPl0rSn9h2Ty5xu5zUosSkojDQf/6Jaz9KFFbM7" + `
        "CLj8fh2YBRZyIa01XwXsgSKMHTwcsDfiyyg0ldbJe03TgrCdlWrJx93CGESOSOmAItw4cxgMUaqPqtv8D3pYN" + `
        "fU1MvfVSqjbhLU1VZMmKQwDpgG/zMzZkD1XwHXd9cT1MTYKXGrSmVykbtSiMKMCaOAn0GKko4J3TufQQVigEk" + `
        "9crsyAcazVaWRZTH3Vl+NMhghcQkwAg/GaaNu74nDp3EF1CQMmOurhckx64goqltpeDaQeQM/QuTPD0RT1Y/E" + `
        "4pxu2HI06L1aDUMr/ETJ78mF53umH79WGWWd/zLXvTRLLM/CpmJeYBcg7oPlF62kGab9hP3a8bPn7rfuWhKls" + `
        "etu66jzyuikKmvTODf+pt9IVDkvXnrI80KT+QXPeV0lXl4PIgAAAABJRU5ErkJggg=="
 
 ##################################################################################################
 
 function frmMain_Show {
   Add-Type -AssemblyName System.Windows.Forms
   [Windows.Forms.Application]::EnableVisualStyles()
 
   $ico = [Drawing.Icon]::ExtractAssociatedIcon(($PSHome + '\powershell_ise.exe'))
 
   $cdcp = New-Object CodeDom.Compiler.CompilerParameters
   $dict = New-Object "Collections.Generic.Dictionary[String, String]"
   $dict.Add("CompilerVersion", "v3.5")
 
   $frmMain = New-Object Windows.Forms.Form
   $mnuMain = New-Object Windows.Forms.MenuStrip
   $mnuFile = New-Object Windows.Forms.ToolStripMenuItem
   $mnuNDoc = New-Object Windows.Forms.ToolStripMenuItem
   $mnuOpen = New-Object Windows.Forms.ToolStripMenuItem
   $mnuSave = New-Object Windows.Forms.ToolStripMenuItem
   $mnuQSav = New-Object Windows.Forms.ToolStripMenuItem
   $mnuEmp1 = New-Object Windows.Forms.ToolStripSeparator
   $mnuExit = New-Object Windows.Forms.ToolStripMenuItem
   $mnuEdit = New-Object Windows.Forms.ToolStripMenuItem
   $mnuUndo = New-Object Windows.Forms.ToolStripMenuItem
   $mnuRedo = New-Object Windows.Forms.ToolStripMenuItem
   $mnuEmp2 = New-Object Windows.Forms.ToolStripSeparator
   $mnuCopy = New-Object Windows.Forms.ToolStripMenuItem
   $mnuPast = New-Object Windows.Forms.ToolStripMenuItem
   $mnuICut = New-Object Windows.Forms.ToolStripMenuItem
   $mnuEmp3 = New-Object Windows.Forms.ToolStripSeparator
   $mnuSAll = New-Object Windows.Forms.ToolStripMenuItem
   $mnuITag = New-Object Windows.Forms.ToolStripMenuItem
   $mnuView = New-Object Windows.Forms.ToolStripMenuItem
   $mnuFont = New-Object Windows.Forms.ToolStripMenuItem
   $mnuEmp4 = New-Object Windows.Forms.ToolStripSeparator
   $mnuOpac = New-Object Windows.Forms.ToolStripMenuItem
   $mnuOp50 = New-Object Windows.Forms.ToolStripMenuItem
   $mnuOp60 = New-Object Windows.Forms.ToolStripMenuItem
   $mnuOp70 = New-Object Windows.Forms.ToolStripMenuItem
   $mnuOp80 = New-Object Windows.Forms.ToolStripMenuItem
   $mnuOp90 = New-Object Windows.Forms.ToolStripMenuItem
   $mnuOpaF = New-Object Windows.Forms.ToolStripMenuItem
   $mnuTgls = New-Object Windows.Forms.ToolStripMenuItem
   $mnuWrap = New-Object Windows.Forms.ToolStripMenuItem
   $mnuPane = New-Object Windows.Forms.ToolStripMenuItem
   $mnuSBar = New-Object Windows.Forms.ToolStripMenuItem
   $mnuMake = New-Object Windows.Forms.ToolStripMenuItem
   $mnuBAsm = New-Object Windows.Forms.ToolStripMenuItem
   $mnuBnRA = New-Object Windows.Forms.ToolStripMenuItem
   $mnuHelp = New-Object Windows.Forms.ToolStripMenuItem
   $mnuInfo = New-Object Windows.Forms.ToolStripMenuItem
   $tsTools = New-Object Windows.Forms.ToolStrip
   $tsBut_1 = New-Object Windows.Forms.ToolStripButton
   $tsBut_2 = New-Object Windows.Forms.ToolStripButton
   $tsBut_3 = New-Object Windows.Forms.ToolStripButton
   $tsBut_4 = New-Object Windows.Forms.ToolStripButton
   $tsSep_1 = New-Object Windows.Forms.ToolStripSeparator
   $tsBut_5 = New-Object Windows.Forms.ToolStripButton
   $tsBut_6 = New-Object Windows.Forms.ToolStripButton
   $tsSep_2 = New-Object Windows.Forms.ToolStripSeparator
   $tsBut_7 = New-Object Windows.Forms.ToolStripButton
   $tsBut_8 = New-Object Windows.Forms.ToolStripButton
   $tsBut_9 = New-Object Windows.Forms.ToolStripButton
   $tsSep_3 = New-Object Windows.Forms.ToolStripSeparator
   $tsBut10 = New-Object Windows.Forms.ToolStripButton
   $tsBut11 = New-Object Windows.Forms.ToolStripButton
   $tsSep_4 = New-Object Windows.Forms.ToolStripSeparator
   $tsLab_1 = New-Object Windows.Forms.ToolStripLabel
   $tsCom_1 = New-Object Windows.Forms.ToolStripComboBox
   $tsLab_2 = New-Object Windows.Forms.ToolStripLabel
   $tsCom_2 = New-Object Windows.Forms.ToolStripComboBox
   $scSplit = New-Object Windows.Forms.SplitContainer
   $tcSpace = New-Object Windows.Forms.TabControl
   $tpBasic = New-Object Windows.Forms.TabPage
   $rtbEdit = New-Object Windows.Forms.RichTextBox
   $tcBuild = New-Object Windows.Forms.TabControl
   $tpError = New-Object Windows.Forms.TabPage
   $lstBugs = New-Object Windows.Forms.ListView
   $chPoint = New-Object Windows.Forms.ColumnHeader
   $chError = New-Object Windows.Forms.ColumnHeader
   $chCause = New-Object Windows.Forms.ColumnHeader
   $tpBuild = New-Object Windows.Forms.TabPage
   $lblName = New-Object Windows.Forms.Label
   $txtName = New-Object Windows.Forms.TextBox
   $chkIDbg = New-Object Windows.Forms.CheckBox
   $chkIMem = New-Object Windows.Forms.CheckBox
   $tpRefer = New-Object Windows.Forms.TabPage
   $lboRefs = New-Object Windows.Forms.ListBox
   $mnuRefs = New-Object Windows.Forms.ContextMenuStrip
   $mnuIMov = New-Object Windows.Forms.ToolStripMenuItem
   $mnuICnM = New-Object Windows.Forms.ToolStripMenuItem
   $mnuIBuf = New-Object Windows.Forms.ToolStripMenuItem
   $mnuIIns = New-Object Windows.Forms.ToolStripMenuItem
   $imgList = New-Object Windows.Forms.ImageList
   $sbPanel = New-Object Windows.Forms.StatusBar
   $sbPnl_1 = New-Object Windows.Forms.StatusBarPanel
   $sbPnl_2 = New-Object Windows.Forms.StatusBarPanel
   $sbPnl_3 = New-Object Windows.Forms.StatusBarPanel
   #
   #
   $mnuMain.Items.AddRange(@($mnuFile, $mnuEdit, $mnuView, $mnuMake, $mnuHelp))
   #
   #
   $mnuFile.DropDownItems.AddRange(@($mnuNDoc, $mnuOpen, $mnuSave, $mnuQSav, $mnuEmp1, $mnuExit))
   $mnuFile.Text = "&File"
   #
   #
   $mnuNDoc.ShortcutKeys = "Control", "N"
   $mnuNDoc.Text = "&New"
   $mnuNDoc.Add_Click({Watch-UnsavedData; $script:src = $null})
   #
   #
   $mnuOpen.ShortcutKeys = "Control", "O"
   $mnuOpen.Text = "&Open..."
   $mnuOpen.Add_Click({Open-Document})
   #
   #
   $mnuSave.ShortcutKeys = "Control", "S"
   $mnuSave.Text = "&Save"
   $mnuSave.Add_Click({Save-Document})
   #
   #
   $mnuQSav.ShortcutKeys = "F2"
   $mnuQSav.Text = "&Quick Save"
   $mnuQSav.Add_Click({Save-DocumentQuickly})
   #
   #
   $mnuExit.ShortcutKeys = "Control", "X"
   $mnuExit.Text = "E&xit"
   $mnuExit.Add_Click({$frmMain.Close()})
   #
   #
   $mnuEdit.DropDownItems.AddRange(@($mnuUndo, $mnuRedo, $mnuEmp2, $mnuCopy, $mnuPast, $mnuICut, `
                                                                    $mnuEmp3, $mnuSAll, $mnuITag))
   $mnuEdit.Text = "&Edit"
   #
   #
   $mnuUndo.ShortcutKeys = "Control", "Z"
   $mnuUndo.Text = "&Undo"
   $mnuUndo.Add_Click({$rtbEdit.Undo()})
   #
   #
   $mnuRedo.ShortcutKeys = "Control", "D"
   $mnuRedo.Text = "&Redo"
   $mnuRedo.Add_Click({$rtbEdit.Redo()})
   #
   #
   $mnuCopy.ShortcutKeys = "Control", "C"
   $mnuCopy.Text = "&Copy"
   $mnuCopy.Add_Click({if ($rtbEdit.SelectionLength -ge 0) {$rtbEdit.Copy()}})
   #
   #
   $mnuPast.ShortcutKeys = "Control", "V"
   $mnuPast.Text = "&Paste"
   $mnuPast.Add_Click({$rtbEdit.Paste()})
   #
   #
   $mnuICut.ShortcutKeys = "Del"
   $mnuICut.Text = "Cut &Item"
   $mnuICut.Add_Click({if ($rtbEdit.SelectionLength -ge 0) {$rtbEdit.Cut()}})
   #
   #
   $mnuSAll.ShortcutKeys = "Control", "A"
   $mnuSAll.Text = "Select &All"
   $mnuSAll.Add_Click({$rtbEdit.SelectAll()})
   #
   #
   $mnuITag.ShortcutKeys = "F3"
   $mnuITag.Text = "Insert &Tag"
   $mnuITag.Add_Click($mnuITag_Click)
   #
   #
   $mnuView.DropDownItems.AddRange(@($mnuFont, $mnuEmp4, $mnuOpac, $mnuTgls))
   $mnuView.Text = "&View"
   #
   #
   $mnuFont.Text = "&Font..."
   $mnuFont.Add_Click($mnuFont_Click)
   #
   #
   $mnuOpac.DropDownItems.AddRange(@($mnuOp50, $mnuOp60, $mnuOp70, $mnuOp80, $mnuOp90, $mnuOpaF))
   $mnuOpac.Text = "&Opacity"
   #
   #
   $mnuOp50.Text = "50%"
   $mnuOp50.Add_Click({Set-Opacity $mnuOp50; $ops = $mnuOp50})
   #
   #
   $mnuOp60.Text = "60%"
   $mnuOp60.Add_Click({Set-Opacity $mnuOp60; $ops = $mnuOp60})
   #
   #
   $mnuOp70.Text = "70%"
   $mnuOp70.Add_Click({Set-Opacity $mnuOp70; $ops = $mnuOp70})
   #
   #
   $mnuOp80.Text = "80%"
   $mnuOp80.Add_Click({Set-Opacity $mnuOp80; $ops = $mnuOp80})
   #
   #
   $mnuOp90.Text = "90%"
   $mnuOp90.Add_Click({Set-Opacity $mnuOp90; $ops = $mnuOp90})
   #
   #
   $ops = $mnuOpaF
   $mnuOpaF.Checked = $true
   $mnuOpaF.Text = "Opaque"
   $mnuOpaF.Add_Click($mnuOpaF_Click)
   #
   #
   $mnuTgls.DropDownItems.AddRange(@($mnuWrap, $mnuPane, $mnuSBar))
   $mnuTgls.Text = "&Toggles"
   #
   #
   $mnuWrap.Checked = $true
   $mnuWrap.ShortcutKeys = "Control", "W"
   $mnuWrap.Text = "&Wrap Mode"
   $mnuWrap.Add_Click($mnuWrap_Click)
   #
   #
   $mnuPane.Checked = $true
   $mnuPane.ShortcutKeys = "Control", "L"
   $mnuPane.Text = "&Lower Pane"
   $mnuPane.Add_Click($mnuPane_Click)
   #
   #
   $mnuSBar.Checked = $true
   $mnuSBar.ShortcutKeys = "Control", "B"
   $mnuSBar.Text = "Status &Bar"
   $mnuSBar.Add_Click($mnuSBar_Click)
   #
   #
   $mnuMake.DropDownItems.AddRange(@($mnuBAsm, $mnuBnRA))
   $mnuMake.Text = "&Build"
   #
   #
   $mnuBAsm.ShortcutKeys = "F5"
   $mnuBAsm.Text = "&Compile"
   $mnuBAsm.Add_Click({Invoke-Builder})
   #
   #
   $mnuBnRA.ShortcutKeys = "F9"
   $mnuBnRA.Text = "Compile And &Run"
   $mnuBnRA.Add_Click({Start-AfterBuilding})
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
   $tsTools.ImageList = $imgList
   $tsTools.Items.AddRange(@($tsBut_1, $tsBut_2, $tsBut_3, $tsBut_4, $tsSep_1, $tsBut_5, `
         $tsBut_6, $tsSep_2, $tsBut_7, $tsBut_8, $tsBut_9, $tsSep_3, $tsBut10, $tsBut11, `
         $tsSep_4, $tsLab_1, $tsCom_1, $tsLab_2, $tsCom_2))
   #
   #
   $tsBut_1.ImageIndex = 3
   $tsBut_1.ToolTipText = "New"
   $tsBut_1.Add_Click({Watch-UnsavedData; $script:src = $null})
   #
   #
   $tsBut_2.ImageIndex = 4
   $tsBut_2.ToolTipText = "Open"
   $tsBut_2.Add_Click({Open-Document})
   #
   #
   $tsBut_3.ImageIndex = 5
   $tsBut_3.ToolTipText = "Save"
   $tsBut_3.Add_Click({Save-Document})
   #
   #
   $tsBut_4.ImageIndex = 6
   $tsBut_4.ToolTipText = "Quick Save"
   $tsBut_4.Add_Click({Save-DocumentQuickly})
   #
   #
   $tsBut_5.ImageIndex = 7
   $tsBut_5.ToolTipText = "Undo"
   $tsBut_5.Add_Click({$rtbEdit.Undo()})
   #
   #
   $tsBut_6.ImageIndex = 8
   $tsBut_6.ToolTipText = "Redo"
   $tsBut_6.Add_Click({$rtbEdit.Redo()})
   #
   #
   $tsBut_7.ImageIndex = 9
   $tsBut_7.ToolTipText = "Copy"
   $tsBut_7.Add_Click({if ($rtbEdit.SelectionLength -ge 0) {$rtbEdit.Copy()}})
   #
   #
   $tsBut_8.ImageIndex = 10
   $tsBut_8.ToolTipText = "Paste"
   $tsBut_8.Add_Click({$rtbEdit.Paste()})
   #
   #
   $tsBut_9.ImageIndex = 11
   $tsBut_9.ToolTipText = "Cut Item"
   $tsBut_9.Add_Click({if ($rtbEdit.SelectionLength -ge 0) {$rtbEdit.Cut()}})
   #
   #
   $tsBut10.ImageIndex = 12
   $tsBut10.ToolTipText = "Build"
   $tsBut10.Add_Click({Invoke-Builder})
   #
   #
   $tsBut11.ImageIndex = 13
   $tsBut11.ToolTipText = "Build And Run"
   $tsBut11.Add_Click({Start-AfterBuilding})
   #
   #
   $tsLab_1.Text = ".NET:"
   #
   #
   $tsCom_1.Items.AddRange(@("v2.0", "v3.5"))
   $tsCom_1.SelectedItem = "v2.0"
   $tsCom_1.Add_SelectedIndexChanged($tsCom_1_SelectedIndexChanged)
   #
   #
   $tsLab_2.Text = "Type:"
   #
   #
   $tsCom_2.Items.AddRange(@("Console", "Library", "WinForms"))
   $tsCom_2.SelectedItem = "Console"
   $tsCom_2.Add_SelectedIndexChanged($tsCom_2_SelectedIndexChanged)
   #
   #
   $scSplit.Dock = "Fill"
   $scSplit.Panel1.Controls.Add($tcSpace)
   $scSplit.Panel2.Controls.Add($tcBuild)
   $scSplit.Panel2MinSize = 17
   $scSplit.Orientation = "Horizontal"
   $scSplit.SplitterDistance = 330
   #
   #
   $tcSpace.Controls.Add($tpBasic)
   $tcSpace.Dock = "Fill"
   $tcSpace.ImageList = $imgList
   #
   #
   $tpBasic.Controls.Add($rtbEdit)
   $tpBasic.ImageIndex = 0
   $tpBasic.Text = "Untitled"
   $tpBasic.UseVisualStyleBackColor = $true
   #
   #
   $rtbEdit.AcceptsTab = $true
   $rtbEdit.Dock = "Fill"
   $rtbEdit.Font = New-Object Drawing.Font("Courier New", 10)
   $rtbEdit.ScrollBars = "Both"
   $rtbEdit.Add_Click({Write-CursorPoint})
   $rtbEdit.Add_KeyUp({Write-CursorPoint})
   $rtbEdit.Add_TextChanged($rtbEdit_TextChanged)
   #
   #
   $tcBuild.Controls.AddRange(@($tpError, $tpBuild, $tpRefer))
   $tcBuild.Dock = "Fill"
   #
   #
   $tpError.Controls.Add($lstBugs)
   $tpError.Text = "Errors"
   $tpError.UseVisualStyleBackColor = $true
   #
   #
   $lstBugs.Columns.AddRange(@($chPoint, $chError, $chCause))
   $lstBugs.Dock = "Fill"
   $lstBugs.FullRowSelect = $true
   $lstBugs.GridLines = $true
   $lstBugs.MultiSelect = $false
   $lstBugs.ShowItemToolTips = $true
   $lstBugs.SmallImageList = $imgList
   $lstBugs.View = "Details"
   #
   #
   $chPoint.Text = "Line"
   $chPoint.Width = 50
   #
   #
   $chError.Text = "Error"
   $chError.TextAlign = "Right"
   $chError.Width = 65
   #
   #
   $chCause.Text = "Description"
   $chCause.Width = 648
   #
   #
   $tpBuild.Controls.AddRange(@($lblName, $txtName, $chkIDbg, $chkIMem))
   $tpBuild.Text = "Output"
   $tpBuild.UseVisualStyleBackColor = $true
   #
   #
   $lblName.Location = New-Object Drawing.Point(8, 8)
   $lblName.Size = New-Object Drawing.Size(57, 17)
   $lblName.Text = "Assembly:"
   #
   #
   $txtName.Anchor = "Left, Top, Right"
   $txtName.Location = New-Object Drawing.Point(73, 7)
   $txtName.Width = 113
   #
   #
   $chkIDbg.Checked = $true
   $chkIDbg.Location = New-Object Drawing.Point(23, 29)
   $chkIDbg.Text = "Include Debug Info"
   $chkIDbg.Width = 120
   #
   #
   $chkIMem.Enabled = $false
   $chkIMem.Location = New-Object Drawing.Point(23, 53)
   $chkIMem.Text = "Build In Memory"
   $chkIMem.Width = 130
   $chkIMem.Add_Click($chkIMem_Click)
   #
   #
   $tpRefer.Controls.Add($lboRefs)
   $tpRefer.Text = "References"
   $tpRefer.UseVisualStyleBackColor = $true
   #
   #
   $lboRefs.ContextMenuStrip = $mnuRefs
   $lboRefs.Dock = "Fill"
   $lboRefs.SelectionMode = "One"
   #
   #
   $mnuRefs.Items.AddRange(@($mnuIMov, $mnuICnM, $mnuIBuf, $mnuIIns))
   #
   #
   $mnuIMov.ShortcutKeys = "Del"
   $mnuIMov.Text = "Remove Item"
   $mnuIMov.Add_Click({$lboRefs.Items.Remove($lboRefs.SelectedItem)})
   #
   #
   $mnuICnM.ShortcutKeys = "Control", "M"
   $mnuICnM.Text = "Copy And Move"
   $mnuICnM.Add_Click($mnuICnM_Click)
   #
   #
   $mnuIBuf.Text = "Insert Copied..."
   $mnuIBuf.Add_Click({if ($script:buf -ne $null) {$lboRefs.Items.Add($script:buf)}})
   #
   #
   $mnuIIns.Text = "Add Reference"
   $mnuIIns.Add_Click($mnuIIns_Click)
   #
   #
   $i_1, $i_2, $i_3, $i_4, $i_5, $i_6, $i_7, $i_8, $i_9, $i10, $i11, $i12, $i13, `
                        $i14, $i15, $i16 | % {$imgList.Images.Add((Get-Image $_))}
   #
   #
   $sbPanel.Panels.AddRange(@($sbPnl_1, $sbPnl_2, $sbPnl_3))
   $sbPanel.ShowPanels = $true
   $sbPanel.SizingGrip = $false
   #
   #
   $sbPnl_1.AutoSize = "Spring"
   #
   #
   $sbPnl_2.Alignment = "Center"
   $sbPnl_2.AutoSize = "Contents"
   #
   #
   $sbPnl_3.Width = 73
   #
   #
   $frmMain.ClientSize = New-Object Drawing.Size(790, 590)
   $frmMain.Controls.AddRange(@($scSplit, $sbPanel, $tsTools, $mnuMain))
   $frmMain.FormBorderStyle = "FixedSingle"
   $frmMain.Icon = $ico
   $frmMain.MainMenuStrip = $mnuMain
   $frmMain.StartPosition = "CenterScreen"
   $frmMain.Text = "Snippet Compiler"
   $frmMain.Add_Load($frmMain_Load)
 
   [void]$frmMain.ShowDialog()
 }
 
 ##################################################################################################
 
 function frmInfo_Show {
   $frmInfo = New-Object Windows.Forms.Form
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
   $lblName.Font = New-Object Drawing.Font("Microsoft Sans Serif", 8, [Drawing.FontStyle]::Bold)
   $lblName.Location = New-Object Drawing.Point(53, 19)
   $lblName.Size = New-Object Drawing.Size(360, 18)
   $lblName.Text = "Snippet Compiler v3.00"
   #
   #
   $lblCopy.Location = New-Object Drawing.Point(67, 37)
   $lblCopy.Size = New-Object Drawing.Size(360, 23)
   $lblCopy.Text = "(C) 2012-2013 greg zakharov gregzakh@gmail.com"
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
   $frmInfo.ShowInTaskBar = $false
   $frmInfo.StartPosition = "CenterScreen"
   $frmInfo.Text = "About..."
   $frmInfo.Add_Load({$pbImage.Image = $ico.ToBitmap()})
 
   [void]$frmInfo.ShowDialog()
 }
 
 ##################################################################################################
 
 frmMain_Show
`

