---
Author: gyorgy nemesmagasi
Publisher: 
Copyright: â© 2010 . all rights reserved.
Email: 
Version: 2.2.0.1358
Encoding: utf-8
License: cc0
PoshCode ID: 4268
Published Date: 2013-06-26t15
Archived Date: 2013-06-29t06
---

# add-on.helpbrowser.psm1 - 

## Description

this file was uploaded by a powergui script editor add-on.

## Comments



## Usage

to load this module in your script editor 1. open the script editor.2. select "powershell libraries" from the file menu.3. check the add-on.helpbrowser module. 4. click on ok to close the "powershell libraries" dialog.  alternatively you can load the module from the embedded console by invoking this. import-module -name add-on.helpbrowser  please provide feedback on the powergui forums.                                                   #

## TODO



## module

``

## Code

`
 Set-StrictMode -Version 2
 
 
 
 if ($Host.Name â��ne 'PowerGUIScriptEditorHost') { return }
 if ($Host.Version -lt '2.2.0.1358') {
 	[System.Windows.Forms.MessageBox]::Show("The ""$(Split-Path -Path $PSScriptRoot -Leaf)"" Add-on module requires version 2.2.0.1358 or later of the Script Editor. The current Script Editor version is $($Host.Version).$([System.Environment]::NewLine * 2)Please upgrade to version 2.2.0.1358 and try again.","Version 2.2.0.1358 or later is required",[System.Windows.Forms.MessageBoxButtons]::OK,[System.Windows.Forms.MessageBoxIcon]::Information) | Out-Null
 	return
 }
 
 $pgse = [Quest.PowerGUI.SDK.ScriptEditorFactory]::CurrentInstance
 
 
 
 $HelpBrowserIconLibrary = @{
 }
 
 $HelpBrowserImageLibrary = @{
 }
 
 $HelpIcon = New-Object System.Drawing.Icon -ArgumentList "$PSScriptRoot\Resources\Help.ico",16,16
 $AliasIcon = New-Object System.Drawing.Icon -ArgumentList "$PSScriptRoot\Resources\Alias.ico",16,16
 $CmdletIcon = New-Object System.Drawing.Icon -ArgumentList "$PSScriptRoot\Resources\Cmdlet.ico",16,16
 
 $HelpBrowserImageList = New-Object -TypeName System.Windows.Forms.ImageList
 $HelpBrowserImageList.Images.Add('HelpIcon', $HelpIcon.ToBitmap())
 $HelpBrowserImageList.Images.Add('AliasIcon', $AliasIcon.ToBitmap())
 $HelpBrowserImageList.Images.Add('CmdletIcon', $CmdletIcon.ToBitmap())
 
 
 
 $PreLoadHelpText = $false
 $HelpPanelHeigh = 500
 $HelpPanelWidth = 430
 $HelpTextWidth = 80
 
 
 
 	function AddNode {
 		param(
 			$Parent,
 			$NodeName,
 			$NodeText,
 			$NodeImageIndex
 		)
 		$script:node = New-Object -TypeName System.Windows.Forms.TreeNode
 		$node.Name = $NodeName
 		$node.Text = $NodeText
 		$node.ImageIndex = $NodeImageIndex
 		$node.SelectedImageIndex = $NodeImageIndex
 		$Parent.Nodes.Add($node)
 	}
 	
 
 
 $helpBrowser = New-Object System.Windows.Forms.TreeView
 $helpBrowser.ImageList = $HelpBrowserImageList
 
 AddNode $helpBrowser 'About' 'About' 0
 Get-Help about | ForEach-Object { AddNode $helpBrowser.Nodes[0] $_.Name $_.Name 0 }
 
 AddNode $helpBrowser 'Alias' 'Alias' 1
 Get-Alias | ForEach-Object {
   AddNode $helpBrowser.Nodes[1] $_.Name ($_.Name + '  ('  + $_.ResolvedCommandName + ')')  1
 }
 
 AddNode $helpBrowser  'Cmdlet' 'Cmdlet' 2
 Get-Command -CommandType Cmdlet | ForEach-Object {
 	$CmdNodeText = $_.Name
 	if (get-alias -Definition $_.Name -ErrorAction SilentlyContinue) {
 		$CmdNodeText  +=  "  ("+ [string]::join(" ,", (get-alias -Definition $_.Name -ErrorAction SilentlyContinue | ForEach-Object {$_.Name}) ) + ")"	
 	}	
 	AddNode $helpBrowser.Nodes[2] $_.Name $CmdNodeText 2
 }
 
 $helpBrowser.Add_NodeMouseClick( [System.Windows.Forms.TreeNodeMouseClickEventHandler] { param($sender, $e)
 	if (($e.Button -ne [System.Windows.Forms.MouseButtons]::Left) -or
 		 ($e.Node.Level -ne 1)) {
 		return
 	}	
 	
 	if ($e.Node.Tag -eq $null) {
 		if ($e.Node.Parent.Name -eq 'About') {
 			$e.Node.Tag = Get-Help $e.Node.Name  | Out-String -Width $HelpTextWidth
 		}
 		else {
 			#$e.Node.Tag = Get-Help $e.Node.Name  | Out-String -Width 200
 			powershell.exe -OutputFormat Text -command "Get-Help -Name $($e.Node.Name) -Full | Out-File -Width $($HelpTextWidth) $Env:temp\HelpText.txt"
 			$e.Node.Tag = [System.String]::Join("`n", (Get-Content -ReadCount 0 "$Env:TEMP\HelpText.txt"))				
 		}
 	}
 	$HelpWindow.Control.Text = $e.Node.Tag
 
 })
 
 $helpBrowser.add_NodeMouseDoubleClick( [System.Windows.Forms.TreeNodeMouseClickEventHandler] { param($sender, $e)
 	if (($e.Button -ne [System.Windows.Forms.MouseButtons]::Left) -or 
 		($e.Node.Level -ne 1) -or ($e.Node.Parent.Name -eq 'About')  -or ($pgSE.CurrentDocumentWindow -eq $null)) {
 		return
 	}
 	
 	if (($originalWindowProperty = $pgSE.CurrentDocumentWindow.GetType().GetProperty('OriginalWindow',[System.Reflection.BindingFlags]'NonPublic,Instance')) -and
 	    ($originalWindow = $originalWindowProperty.GetValue($pgSE.CurrentDocumentWindow,$null)) -and
 	    ($originalWindow | Get-Member -MemberType Property -Name PSControl -ErrorAction SilentlyContinue) -and
 	    ($scriptEditorControl = $originalWindow.PSControl) -and 
 		($scriptEditorControl.GetType().BaseType.FullName -eq 'ActiproSoftware.SyntaxEditor.SyntaxEditor')) {
 			$scriptEditorControl.Document.InsertText([ActiproSoftware.SyntaxEditor.DocumentModificationType]::Typing,$scriptEditorControl.Caret.Offset, $e.Node.Name)
 	}
 	
 })
 
 if ($PreLoadHelpText) {
 	$helpBrowser.Nodes[0].Nodes | ForEach-Object {$_.Tag = Get-Help $_.Name -Full | Out-String -Width $HelpTextWidth }
 	$helpBrowser.Nodes[1].Nodes | ForEach-Object {$_.Tag = Get-Help $_.Name -Full | Out-String -Width $HelpTextWidth }
 	$helpBrowser.Nodes[2].Nodes | ForEach-Object {$_.Tag = Get-Help $_.Name -Full | Out-String -Width $HelpTextWidth }
 }
 
 
 
 if (-not ($HelpBrowserWindow = $pgse.ToolWindows['HelpBrowser'])) {
 	$HelpBrowserWindow = $pgse.ToolWindows.Add('HelpBrowser')
 	$HelpBrowserWindow.Title = '&Help Browser' -replace '&'
 	$HelpBrowserWindow.Control = $helpBrowser
 	$HelpBrowserWindow.Control.Parent.CanBecomeDocument = [ActiproSoftware.ComponentModel.DefaultableBoolean]::False
 	$HelpBrowserWindow.Control.Parent.Image = $HelpIcon.ToBitmap()	
 	$HelpBrowserWindow.Visible = $true
 } 
 else {
 		$HelpBrowserWindow.Control = $helpBrowser
 }
 
 
 
 if (-not ($goToHelpBrowserCommand = $pgse.Commands['GoCommand.HelpBrowser'])) {
 	$goToHelpBrowserCommand = New-Object -TypeName Quest.PowerGUI.SDK.ItemCommand -ArgumentList 'GoCommand', 'HelpBrowser'
 	$goToHelpBrowserCommand.Text = '&Help Browser'
 	$goToHelpBrowserCommand.Image = $HelpIcon.ToBitmap()
 	if ($goMenu = $pgse.Menus['MenuBar.Go']) {
 		$index = $goMenu.Items.Count + 1
 		if ($index -lt 10) {
 			$goToHelpBrowserCommand.AddShortcut("Ctrl+${index}")
 		}
 	}
 	$goToHelpBrowserCommand.ScriptBlock = {
 		$pgse = [Quest.PowerGUI.SDK.ScriptEditorFactory]::CurrentInstance
 		if ($HelpBrowserWindow = $pgse.ToolWindows['HelpBrowser']) {
 			$HelpBrowserWindow.Visible = $true
 			$HelpBrowserWindow.Control.Invoke([EventHandler]{$HelpBrowserWindow.Control.Parent.Activate($true)})
 		}
 	}
 
 	$pgse.Commands.Add($goToHelpBrowserCommand)
 }
 
 
 
 if ($goMenu = $pgse.Menus['MenuBar.Go']) {
 	$goMenu.Items.Add($goToHelpBrowserCommand)
 }
 
 
 
 
 
 	$HelpControl = New-Object System.Windows.Forms.RichTextBox
 	$HelpControl.Text = @"
 Help browser
 
 created by Gyorgy Nemesmagasi
 Â© 2010 . All rights reserved.
 "@
 	
 
 
 	if (-not ($HelpWindow = $pgse.ToolWindows['Help'])) {
 		$HelpWindow = $pgse.ToolWindows.Add('Help')
 		$HelpWindow.Title = '&Help' -replace '&'
 		$HelpWindow.Control = $HelpControl
 		$HelpWindow.Visible = $true
 		$HelpWindow.Control.Parent.Image = $HelpIcon.ToBitmap()
 		$HelpWindow.Control.Invoke([EventHandler]{$HelpWindow.Control.Parent.State = 'Floating'})
 		$HelpWindow.Control.Invoke([EventHandler]{$HelpWindow.Control.Parent.FloatingSize = New-Object system.Drawing.Size($HelpPanelWidth,$HelpPanelHeigh)})
 		$moveHeigh = [System.Windows.Forms.Screen]::PrimaryScreen.WorkingArea.Height - $HelpPanelHeigh
 		$moveWidth = [System.Windows.Forms.Screen]::PrimaryScreen.WorkingArea.Width - $HelpPanelWidth
 		$HelpWindow.Control.Invoke([EventHandler]{$HelpWindow.Control.Parent.FloatingLocation = New-Object system.Drawing.Size($moveWidth,$moveHeigh)})
 		$HelpWindow.Control.Invoke([EventHandler]{$HelpWindow.Control.Parent.Activate($true)})
 	} else {
 		$HelpWindow.Control = $HelpControl
 	}
 
 
 
 	if (-not ($goToHelpCommand = $pgse.Commands['GoCommand.Help'])) {
 		$goToHelpCommand = New-Object -TypeName Quest.PowerGUI.SDK.ItemCommand -ArgumentList 'GoCommand', 'Help'
 		$goToHelpCommand.Text = '&Help'
 		$goToHelpCommand.Image = $HelpIcon.ToBitmap()
 		if ($goMenu = $pgse.Menus['MenuBar.Go']) {
 			$index = $goMenu.Items.Count + 1
 			if ($index -lt 10) {
 				$goToHelpCommand.AddShortcut("Ctrl+${index}")
 			}
 		}
 		$goToHelpCommand.ScriptBlock = {
 			$pgse = [Quest.PowerGUI.SDK.ScriptEditorFactory]::CurrentInstance
 			if ($HelpWindow = $pgse.ToolWindows['Help']) {
 				$HelpWindow.Visible = $true
 				$HelpWindow.Control.Invoke([EventHandler]{$HelpWindow.Control.Parent.Activate($true)})
 			}
 		}
 
 		$pgse.Commands.Add($goToHelpCommand)
 	}
 
 
 
 	if ($goMenu = $pgse.Menus['MenuBar.Go']) {
 		$goMenu.Items.Add($goToHelpCommand)
 	}
 
 
 
 
 
 
 $ExecutionContext.SessionState.Module.OnRemove = {
 	$pgse = [Quest.PowerGUI.SDK.ScriptEditorFactory]::CurrentInstance
 	
 	
 	if (($goMenu = $pgse.Menus['MenuBar.Go']) -and
 	    ($scriptExplorerCmdItem = $goMenu.Items['GoCommand.HelpBrowser'])) {
 		$goMenu.Items.Remove($scriptExplorerCmdItem) | Out-Null
 	}
 	if ($scriptExplorerCmdItem = $pgse.Commands['GoCommand.HelpBrowser']) {
 		$pgse.Commands.Remove($scriptExplorerCmdItem) | Out-Null
 	}
 	if ($PGScriptExplorer = $pgse.ToolWindows['HelpBrowser']) {
 		$pgse.ToolWindows.Remove($PGScriptExplorer) | Out-Null
 	}
 	
 	
 	if (($goMenu = $pgse.Menus['MenuBar.Go']) -and
 	    ($scriptExplorerCmdItem = $goMenu.Items['GoCommand.Help'])) {
 		$goMenu.Items.Remove($scriptExplorerCmdItem) | Out-Null
 	}
 	if ($scriptExplorerCmdItem = $pgse.Commands['GoCommand.Help']) {
 		$pgse.Commands.Remove($scriptExplorerCmdItem) | Out-Null
 	}
 	if ($PGScriptExplorer = $pgse.ToolWindows['Help']) {
 		$pgse.ToolWindows.Remove($PGScriptExplorer) | Out-Null
 	}
 }
 
`

