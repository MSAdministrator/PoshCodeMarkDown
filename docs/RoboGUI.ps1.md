---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 626
Published Date: 
Archived Date: 2009-08-27t01
---

# robogui - 

## Description

a helper gui to create robocopy commands

## Comments



## Usage



## TODO



## script

`update-commandline`

## Code

`#
 #
 
 
 [system.reflection.assembly]::LoadWithPartialName('system.windows.forms') 
 . .\FormsLib.ps1 
 
 
 $RoboHelp = robocopy /? | Select-String '::' 
 $r = [regex]'(.*)::(.*)' 
 $RoboHelpObject = $RoboHelp | select ` 
     @{Name='Parameter';Expression={ $r.Match( $_).groups[1].value.trim()}}, 
     @{Name='Description';Expression={ $r.Match( $_).groups[2].value.trim()}} 
 
 $RoboHelpObject = $RoboHelpObject |% {$Cat = 'General'} { 
     if ($_.parameter -eq '') { if ($_.Description -ne ''){ 
         $cat = $_.description -Replace 'options :',''} 
     } else { 
         $_ | select @{Name='Category';Expression={$cat}},parameter,description 
     } 
 } 
 
 
 
 $frmRobocopy = new-Object System.Windows.Forms.form 
 set-controlFormat $frmRobocopy ` 
   -Location '{X=46,Y=46}'` 
   -Size '{Width=1223, Height=850}'` 
   -Text 'RoboCopy PowerShell GUI'` 
 
 
 $dtSwitches = new-object Data.datatable 
 
 $Col =  new-object Data.DataColumn 
 $col.DataType = [bool] 
 $Col.ColumnName = "Enabled" 
 
 $dtSwitches.Columns.Add($Col) 
 
 "Category","Name","Description" | 
  Foreach { 
     $Col =  new-object Data.DataColumn 
     $Col.ColumnName = $_ 
    $dtSwitches.Columns.Add($Col) 
   } 
 
 $RoboHelpObject |? { $_.Parameter -match '/[a-z]*$' } | 
  Foreach { 
     $row = $dtswitches.NewRow() 
     $row.Category = $_.Category 
     $row.Name = $_.Parameter 
     $row.Description = $_.Description   
     $dtSwitches.Rows.Add($row) 
   } 
 
 
 $dtParameters = new-object Data.datatable 
 
 $Col =  new-object Data.DataColumn 
 $col.DataType = [bool] 
 $Col.ColumnName = "Enabled" 
 $dtParameters.Columns.Add($Col) 
 
 
 "Category","Name","Value","Description" | 
  Foreach { 
     $Col =  new-object Data.DataColumn 
     $Col.ColumnName = $_ 
    $dtParameters.Columns.Add($Col) 
   } 
 
 $RoboHelpObject |? { $_.Parameter -match '/[a-z]*:' } | 
  Foreach { 
     $row = $dtParameters.NewRow() 
     $row.Category = $_.Category 
     $row.Name = $_.Parameter 
     $row.Description = $_.Description   
     $dtParameters.Rows.Add($row) 
   } 
 
 
 
 "Source", 
 "Destination", 
 "Files", 
 "Options", 
 "CommandLine" |% {$Y = 30}{ 
   $label = New-Object System.Windows.Forms.Label 
   Set-ControlFormat $label ` 
     -Anchor 'Top, Left'` 
     -Dock 'None'` 
     -Location "{X=50,Y=$y}"` 
     -Name "lbl$_"` 
     -Size '{Width=100, Height=23}'` 
     -Text $_ 
 
  $TextBox = New-Object System.Windows.Forms.TextBox 
   Set-ControlFormat $textBox ` 
     -Anchor 'Top, Left'` 
     -Dock 'None'` 
     -Location "{X=150,Y=$Y}"` 
     -Name "txt$_"` 
     -Size '{Width=400, Height=21}' 
 
 
  $FrmRobocopy.Controls.Add($label) 
   $FrmRobocopy.Controls.Add($TextBox) 
   $Y += 25 
 } 
 
 $command = "RoboCopy" 
 
 $FrmRobocopy.Controls['txtCommandLine'].Enabled = $false 
 $FrmRobocopy.Controls['txtOptions'].Enabled = $false 
 $FrmRobocopy.Controls['txtCommandLine'].Text = $command 
 
 $dgSwitches = new-object windows.forms.DataGridView 
 Set-ControlFormat $dgSwitches ` 
   -Dock 'Fill'` 
   -Name 'dgOptions'` 
   -DataSource  $dtSwitches.psObject.baseobject   
 
 $dgOptions = new-object windows.forms.DataGridView 
 Set-ControlFormat $dgOptions ` 
   -Dock 'Fill'` 
   -Name 'dgOptions'` 
   -DataSource $dtParameters.psObject.baseobject 
 
 $scOptions = new-object System.Windows.Forms.SplitContainer 
 Set-ControlFormat $scOptions ` 
   -Name 'scOptions' ` 
   -Location '{X=20,Y=170}'` 
   -Size '{Width=1170, Height=615}'` 
   -SplitterDistance '534'` 
   -Anchor "Top, Bottom, Left, Right" 
 
 $scOptions.Panel1.Controls.Add( $DGswitches ) 
 $scOptions.Panel2.Controls.Add( $dgOptions ) 
 $FrmRobocopy.Controls.Add($scOptions) 
 
 
 $Switches = "$($dtSwitches |? {$_.Enabled -eq $True} |% {$_.name})" 
 
 function Update-Commandline { 
   $Switches = "$($dtSwitches |? {$_.Enabled -eq $True} |% {$_.name})" 
  $Options = "$($dtParameters |? {$_.Enabled -eq $True} |% {$_.name.split(':')[0] + "":$($_.Value)""})" 
  $FrmRobocopy.Controls['txtOptions'].Text = "$options $Switches" 
  $Command = '$robocopy ' ` 
     + $FrmRobocopy.Controls['txtSource'].Text ` 
     + ' ' + $FrmRobocopy.Controls['txtDestination'].Text ` 
     + " " + $FrmRobocopy.Controls['txtFiles'].Text ` 
     + ' ' + $FrmRobocopy.Controls['txtOptions'].Text 
 
   $FrmRobocopy.Controls['txtCommandLine'].Text = $command 
 } 
 
 $btnUpdate = new-Object System.Windows.Forms.Button 
 set-controlFormat $btnUpdate ` 
   -Anchor 'Top, Left'` 
   -Dock 'None'` 
   -Location '{X=560,Y=30}'` 
   -Size '{Width=75, Height=23}'` 
   -Text 'Update' 
 $btnUpdate.add_Click({Update-Commandline}) 
 
 $FrmRobocopy.Controls.Add($btnUpdate) 
 
 
 $FrmRobocopy.Add_Shown({$frmRoboCopy.Activate() 
   $dgSwitches.AutoResizeColumns() 
   $dgOptions.AutoResizeColumns() 
 }) 
 $FrmRobocopy.ShowDialog()
`

