---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5327
Published Date: 
Archived Date: 2014-07-25t21
---

#  - 

## Description

simple script for work with recycle bin

## Comments



## Usage



## TODO



## script

`recycle-item`

## Code

`#
 #
 <#
 This script creates object which correctly works with ntfs streams and reparse points
 Usage samples:
 
 	'.\*.bak' | recycle -WipeOut -> multiplies objects by zero
 	recycle -Path '.\*.ini' -> sends objects to Recycle Bin
 
 	$RecycleBin.Measure() -> returns true size of all items in Recycle Bin
 	$RecycleBin.Clear() -> this is obvious
 	$RecycleBin.List() -> returns friendly items list
 
 More examples:
 
 	$RecycleBin.Measure('*.mkv')
 	$RecycleBin.Clear('*.txt')
 	$RecycleBin.List('*.doc')
 	$RecycleBin.List('film.mkv').Measure()
 	$RecycleBin.List('good.exe').Restore()
 	$RecycleBin.List('virus.exe').Delete()
 
 	$time_to_die = [TimeSpan]::FromDays(20)
 	($RecycleBin.List() | where PermanentState -GE $time_to_die).Delete()
 #>
 
 $shell32_wrapper = @"
 using System;
 using System.ComponentModel;
 using System.Runtime.InteropServices;
 
 namespace shell32 {
 
 	// Put all the variables required for the DLLImports here
 	enum RecycleFlags : uint { SHERB_NOCONFIRMATION = 0x00000001, SHERB_NOPROGRESSUI = 0x00000002, SHERB_NOSOUND = 0x00000004 }
 
 	public static class RecycleBin {
 		[DllImport("Shell32.dll",CharSet=CharSet.Unicode)]
 			internal static extern uint SHEmptyRecycleBin(IntPtr hwnd, string pszRootPath, RecycleFlags dwFlags);
 		[DllImport("Shell32.dll",CharSet=CharSet.Unicode)]
 			internal static extern uint SHUpdateRecycleBinIcon();
 	}
 
 	public class ShellWrapper : IDisposable {
 
 		// Creates a new wrapper for the local machine
 		public ShellWrapper() { }
 
 		// Disposes of this wrapper
 		public void Dispose() {
 			GC.SuppressFinalize(this);
 		}
 
 		// Put public functions here
 		public void Empty() {
 			RecycleBin.SHEmptyRecycleBin(IntPtr.Zero, null, RecycleFlags.SHERB_NOCONFIRMATION | RecycleFlags.SHERB_NOPROGRESSUI | RecycleFlags.SHERB_NOSOUND);
 		}
 
 		public void IconUpdate() {
 			RecycleBin.SHUpdateRecycleBinIcon();
 		}
 
 		// Occurs on destruction of the Wrapper
 		~ShellWrapper() {
 			Dispose();
 		}
 
 	} // Wrapper class
 }
 "@
 
 Add-type -AssemblyName "Microsoft.VisualBasic"
 Add-Type -TypeDefinition $shell32_wrapper
 Function Recycle-Item
 {
 	[CmdletBinding()]
 	Param(
 		[Parameter(Position=0,Mandatory,ValueFromPipeline)][System.Object[]]$Path,
 		[Parameter(Position=1,Mandatory=$false)][switch]$WipeOut
 	)
 
 	Begin
 	{
 		$dialog_mode = [Microsoft.VisualBasic.FileIO.UIOption]::OnlyErrorDialogs
 		$recycle_mode_default = [Microsoft.VisualBasic.FileIO.RecycleOption]::SendToRecycleBin
 		$recycle_mode_wipeout = [Microsoft.VisualBasic.FileIO.RecycleOption]::DeletePermanently
 		$recycle_mode = switch -Exact ($WipeOut) {$true{$recycle_mode_wipeout};Default{$recycle_mode_default}}
 	}
 	Process
 	{
 		$Path | gi -Force -ErrorAction 'SilentlyContinue' | foreach -Process {
 			if ($_.Mode[0] -eq [char]0x64)
 			{
 				"Removing directory: `"$($_.FullName)`"" | Write-Verbose
 				[Microsoft.VisualBasic.FileIO.FileSystem]::DeleteDirectory($_.FullName,$dialog_mode,$recycle_mode)
 			}
 			else
 			{
 				"Removing file: `"$($_.FullName)`"" | Write-Verbose
 				[Microsoft.VisualBasic.FileIO.FileSystem]::DeleteFile($_.FullName,$dialog_mode,$recycle_mode)
 			}
 		}
 	}
 }
 Set-Alias -Name 'recycle' -Value Recycle-Item
 $method = @{}
 $method['List'] = {
 	.{ param ([Parameter(Mandatory=$false)][string]$Mask)} @args
 
 	$actual_list = $this.Items() | select
 	if ($Mask) { $actual_list = $actual_list | where -Property 'Name' -like -Value $Mask }
 	$sked = @()
 
 	$actual_list | foreach {
 
 		$rec = $_ | select 'Name','Type',
 			@{Name = 'PermanentState'; Expression = { [DateTime]::Now - $_.ModifyDate }},
 			@{Name = 'Object'; Expression = { gi -LiteralPath $_.Path -Force }},
 			@{Name = 'Actions'; Expression = { $_.Verbs() | select }}
 
 		$rec | Add-Member -MemberType 'ScriptMethod' -Name 'RemoveInfoFile' -Value {
 			$t = $this.Object
 			"$($t.PSParentPath)\$($t.PSChildName.Replace('$R','$I'))" | ri 
 		}
 
 		$rec | Add-Member -MemberType 'ScriptMethod' -Name 'Delete' -Value {
 			$this.Object | recycle -WipeOut
 			$this.RemoveInfoFile()
 		}
 
 		$rec | Add-Member -MemberType 'ScriptMethod' -Name 'Restore' -Value {
 			$this.Actions[0].DoIt()
 			if (!$this.Object.Exists) { $this.RemoveInfoFile() }
 		}
 
 		$rec | Add-Member -MemberType 'ScriptMethod' -Name 'Measure' -Value {
 
 			Set-Variable -Name _measure -Value @{
 				Item = $this.Name
 				Links = 0
 				Files = 0
 				Folders = 0
 				Size = 0
 			} -Option AllScope
 
 			function calculate ([System.Object]$Obj, [System.IO.DirectoryInfo]$Fol)
 			{
 				if (-not $Obj) { $Obj = $Fol | ls -Force }
 				if ($Obj)
 				{
 					$Obj | foreach {
 						if ($_.Attributes -match 'ReparsePoint') { ++$_measure.Links }
 						elseif ($_.Attributes -notmatch 'Directory')
 						{
 							Try { $_measure.Size += ($_ | gi -Force -Stream '*' | measure Length -Sum).Sum }
 							Catch { $_measure.Size += $_.Length }
 							Finally { ++$_measure.Files }
 						}
 						else { ++$_measure.Folders; calculate -Fol $_ }
 					}
 				}
 			calculate -Obj $this.Object
 			return New-Object -TypeName 'PSObject' -Property $_measure
 
 		$sked += $rec
 	}
 	return $sked
 }
 $method['Clear'] = {
 	.{ param ([Parameter(Mandatory=$false)][string]$Mask)} @args
 	if ($Mask)
 	{
 		$this.List($Mask) | foreach { $_.Delete() }
 		$this.Wrapper.IconUpdate()
 	}
 	else { $this.Wrapper.Empty() }
 }
 $method['Measure'] = {
 	.{ param ([Parameter(Mandatory=$false)][string]$Mask)} @args
 
 	$res = @{Links = 0; Files = 0; Folders = 0; Size = 0}
 	$this.List($Mask) | foreach {
 		$item = $_.Measure()
 		$res.Links   += $item.Links;   $res.Files += $item.Files
 		$res.Folders += $item.Folders; $res.Size  += $item.Size
 	}
 	return New-Object -TypeName 'PSObject' -Property $res
 }
 
 $bin = (New-Object -ComObject 'Shell.Application').Namespace(0xA)
 $bin | Add-Member -MemberType 'ScriptMethod' -Name 'List' -Value $method.List
 $bin | Add-Member -MemberType 'ScriptMethod' -Name 'Clear' -Value $method.Clear
 $bin | Add-Member -MemberType 'ScriptMethod' -Name 'Measure' -Value $method.Measure
 $bin | Add-Member -MemberType 'NoteProperty' -Name 'Wrapper' -Value (New-Object -TypeName 'shell32.ShellWrapper')
 New-Variable -Name RecycleBin -Value $bin -Option Constant
 Remove-Variable -Name 'method', 'bin', 'shell32_wrapper'
`

