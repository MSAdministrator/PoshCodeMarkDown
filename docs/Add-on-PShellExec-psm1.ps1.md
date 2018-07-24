---
Author: les papier
Publisher: infospectrum inc.
Copyright: â© 2011 infospectrum. all rights reserved.
Email: 
Version: 2.3.0.1503
Encoding: utf-8
License: cc0
PoshCode ID: 4269
Published Date: 2013-06-26t15
Archived Date: 2013-06-29t07
---

# add-on.pshellexec.psm1 - 

## Description

this file was uploaded by a powergui script editor add-on.

## Comments



## Usage

to load this module in your script editor 1. open the script editor.  2. select "powershell libraries" from the file menu.  3. check the add-on.pshellexec module.  4. click on ok to close the "powershell libraries" dialog. 5. download the latest pshellexec version from http

## TODO



## module

``

## Code

`
 Set-StrictMode -Version 2
 
 
 if ($Host.Name â��ne 'PowerGUIScriptEditorHost') { return }
 if ($Host.Version -lt '2.3.0.1503') {
 	[System.Windows.Forms.MessageBox]::Show("The ""$(Split-Path -Path $PSScriptRoot -Leaf)"" Add-on module requires version 2.3.0.1503 or later of the Script Editor. The current Script Editor version is $($Host.Version).$([System.Environment]::NewLine * 2)Please upgrade to version 2.3.0.1503 and try again.","Version 2.3.0.1503 or later is required",[System.Windows.Forms.MessageBoxButtons]::OK,[System.Windows.Forms.MessageBoxIcon]::Information) | Out-Null
 	return
 }
 
 $pgse = [Quest.PowerGUI.SDK.ScriptEditorFactory]::CurrentInstance
 
 
 
 $iconLibrary = @{	
 	PShellExecIcon16 = New-Object System.Drawing.Icon -ArgumentList "$PSScriptRoot\Resources\secure.ico",16,16
 }
 
 $imageLibrary = @{
 	
 	PShellExecImage16 = $iconLibrary['PShellExecIcon16'].ToBitmap()
 }
 
 
 
 if (-not ($PShellCmd = $pgse.Commands['ToolsCommand.PShellExec'])) 
 {
 	$PShellCmd = New-Object -TypeName Quest.PowerGUI.SDK.ItemCommand -ArgumentList 'ToolsCommand','PShellExec'
 	$PShellCmd.Text = '&Secure Scripts'
 	$PShellCmd.Image = $imageLibrary['PShellExecImage16']
 	$PShellCmd.AddShortcut('Ctrl+Alt+S')
 	$psarg = 'PShellExec.ps1.bin'
 	if ([Quest.PowerGUI.SDK.ScriptEditorFactory]::CurrentInstance.CurrentDocumentWindow.Document.Path) 
 	{
 		$psarg =  '/a:"'+[Quest.PowerGUI.SDK.ScriptEditorFactory]::CurrentInstance.CurrentDocumentWindow.Document.Path +'"'
 	}	
 	$PShellCmd.ScriptBlock = 
 	{
 			$runDir = Split-Path -Path $MyInvocation.MyCommand.Definition -Parent
 			$aFile = (Join-Path $rundir 'PShellExec.ps1.bin')			
 			start-process  $runDir\PShellExec.exe -ArgumentList $aFile,$psarg 
 	}
 	$pgse.Commands.Add($PShellCmd)
 }
 
 
 if (($toolMenu = $pgse.Menus['MenuBar.Tools']) -and
 	(-not ($PShellItem = $toolMenu.Items['ToolsCommand.PShellExec']))) 
 { 
 	$toolMenu.Items.Add($PShellCmd)
 	if ($PShellItem = $toolMenu.Items['ToolsCommand.PShellExec']) 
 		{ $PShellItem.FirstInGroup = $true }
 }
 
 
 $ExecutionContext.SessionState.Module.OnRemove = 
 {
 	$pgse = [Quest.PowerGUI.SDK.ScriptEditorFactory]::CurrentInstance
 	if (($PShellCmd = $pgse.Commands['ToolsCommand.PShellExec'])) 
 		{ $pgse.Commands.Add($PShellCmd) }
 	if (($toolMenu = $pgse.Menus['MenuBar.Tools']) -and
 		(($PShellItem = $toolMenu.Items['ToolsCommand.PShellExec']))) 
 		{ $toolMenu.Items.Remove($PShellItem) }
 }
 
`

