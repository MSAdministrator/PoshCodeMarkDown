---
Author: adam driscoll
Publisher: quest software
Copyright: â© 2011 quest software. all rights reserved.
Email: 
Version: 3.0
Encoding: utf-8
License: cc0
PoshCode ID: 4267
Published Date: 2013-06-26t14
Archived Date: 2013-06-29t07
---

# add-on.installportable.p - 

## Description

this file was uploaded by a powergui script editor add-on.

## Comments



## Usage

to load this module in your script editor 1. open the script editor. 2. select "powershell libraries" from the file menu.  3. check the add-on.installportable module.  4. click on ok to close the "powershell libraries" dialog.

## TODO



## module

``

## Code

`
 Set-StrictMode -Version 2
 
 
 if (-not ('PowerShellTypeExtensions.Win32Window' -as [System.Type])) {
 	$cSharpCode = @'
 using System;
 
 namespace PowerShellTypeExtensions {
 	public class Win32Window : System.Windows.Forms.IWin32Window
 	{
 		public static Win32Window CurrentWindow {
 			get {
 				return new Win32Window(System.Diagnostics.Process.GetCurrentProcess().MainWindowHandle);
 			}
 		}
 
 		public Win32Window(IntPtr handle) {
 			_hwnd = handle;
 		}
 
 		public IntPtr Handle {
 			get {
 				return _hwnd;
 			}
 		}
 
 		private IntPtr _hwnd;
 	}
 }
 '@
 
 	Add-Type -ReferencedAssemblies System.Windows.Forms -TypeDefinition $cSharpCode
 }
 
 
 
 $minimumSdkVersion = [System.Version]'1.0'
 
 if ($Host.Name â��ne 'PowerGUIScriptEditorHost') {
 	return
 }
 
 if (-not (Get-Variable -Name PGVersionTable -ErrorAction SilentlyContinue)) {
 	[System.Windows.Forms.MessageBox]::Show([PowerShellTypeExtensions.Win32Window]::CurrentWindow,"The ""$(Split-Path -Path $PSScriptRoot -Leaf)"" Add-on module requires PowerGUI 3.0 or later. The current PowerGUI version is $($Host.Version).$([System.Environment]::NewLine * 2)Please upgrade to the latest version of PowerGUI and try again.",'PowerGUI 3.0 or later is required',[System.Windows.Forms.MessageBoxButtons]::OK,[System.Windows.Forms.MessageBoxIcon]::Information) | Out-Null
 	return
 } elseif ($PGVersionTable.SDKVersion -lt $minimumSdkVersion) {
 	[System.Windows.Forms.MessageBox]::Show([PowerShellTypeExtensions.Win32Window]::CurrentWindow,"The ""$(Split-Path -Path $PSScriptRoot -Leaf)"" Add-on module requires version $minimumSdkVersion or later of the PowerGUI Script Editor SDK. The current SDK version is $($PGVersionTable.SDKVersion).$([System.Environment]::NewLine * 2)Please upgrade to the latest version of PowerGUI and try again.","Version $minimumSdkVersion or later of the SDK is required",[System.Windows.Forms.MessageBoxButtons]::OK,[System.Windows.Forms.MessageBoxIcon]::Information) | Out-Null
 	return
 }
 
 
 
 $iconLibrary = @{
 	InstallPortableIcon16 = New-Object System.Drawing.Icon -ArgumentList "$PSScriptRoot\usb.ico",16,16
 	InstallPortableIcon32 = New-Object System.Drawing.Icon -ArgumentList "$PSScriptRoot\usb.ico",32,32
 }
 $imageLibrary = @{
 	InstallPortable16 = $iconLibrary['InstallPortableIcon16'].ToBitmap()
 }
 
 
 if (-not ($clearConsoleCommand = $PGSE.Commands['ToolsCommand.InstallPortableCommand'])) {
 	$InstallPortableCommand = New-Object -TypeName Quest.PowerGUI.SDK.ItemCommand -ArgumentList 'ToolsCommand','InstallPortableCommand'
 	$InstallPortableCommand.Text = '&Install Portable'
 	$InstallPortableCommand.Image = $imageLibrary['InstallPortable16']
 	$InstallPortableCommand.ScriptBlock = {
 	
 		#
 		#
 		$dialog = New-Object System.Windows.Forms.FolderBrowserDialog
 		$dialog.ShowDialog()
 		
 		$destination = $dialog.SelectedPath
 		
 		if ($destination -ne $null -and $destination.LastIndexOf([system.IO.Path]::DirectorySeparatorChar) -lt 0)
 		{
 			$destination = "$destination$([system.IO.Path]::DirectorySeparatorChar)"
 		}
 		
 		if (-not [String]::IsNullOrEmpty($destination))
 		{
 			$i = 0
 			
 			#
 			#
 			Write-Progress -Activity "Installing PowerGUI to USB Key..." -Status  "Processing..." -PercentComplete $i
 			
 			Start-Job -Name "InstallPortable"  -FilePath "$PSScriptRoot\InstallPortable.ps1" -ArgumentList $destination,$PGHome,"PowerGUI"
 			
 			#
 			#
 			while(-not (Test-Path -Path (Join-Path $destination "PowerGUI ScriptEditor_x86.exe")))
 			{
 				Write-Progress -Activity "Installing PowerGUI to USB Key..." -Status  "Processing..." -PercentComplete $i
 				
 				#
 				#
 				$i++
 				if ($i -gt 100)
 				{
 					$i = 0
 				}
 				
 				Start-Sleep -Milliseconds 250
 			}
 			Write-Progress -Activity "Installing PowerGUI to USB Key..." -Status "Processing..." -Completed
 			
 			#
 			#
 			$jobResult = Receive-Job -Name "InstallPortable" 
 			
 			if ($jobResult -ne $null)
 			{
 				throw $jobResult
 			}
 		}
 	}
 
 	$PGSE.Commands.Add($InstallPortableCommand)
 	
 	$ToolBar = $PGSE.Toolbars | Where { $_.Title -eq "Standard" } 
 	$ToolBar.Items.Add($InstallPortableCommand)
 }
 
 
 $ExecutionContext.SessionState.Module.OnRemove = {
 
 	$standardToolbar = $null
 	foreach ($item in $pgse.Toolbars) {
 		if ($item.Title -eq 'Standard') {
 			$standardToolbar = $item
 			break
 		}
 	}
 	if (($standardToolbar) -and
 	    ($newAddonToolbarButton = $standardToolbar.Items['ToolsCommand.InstallPortableCommand'])) {
 		$standardToolbar.Items.Remove($newAddonToolbarButton) | Out-Null
 	}
 
 if (($viewMenu = $PGSE.Menus['MenuBar.View']) -and
     ($installPortableCommand = $viewMenu.Items['ToolsCommand.InstallPortableCommand'])) {
 	$viewMenu.Items.Remove($installPortableCommand) | Out-Null
 	}
 	
 	if ($installPortableCommand =
 		$PGSE.Commands['ToolsCommand.InstallPortableCommand']) {
 		$PGSE.Commands.Remove($installPortableCommand) | Out-Null
 	}
 }
 
`

