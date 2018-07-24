---
Author: shay levy
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 4070
Published Date: 2013-04-04t16
Archived Date: 2016-07-01t08
---

# set-topmost - 

## Description

set applications/windows on top of other windows

## Comments



## Usage



## TODO



## class

`set-topmost`

## Code

`#
 #
 
 $signature = @"
 	
 	[DllImport("user32.dll")]  
 	public static extern IntPtr FindWindow(string lpClassName, string lpWindowName);  
 
 	public static IntPtr FindWindow(string windowName){
 		return FindWindow(null,windowName);
 	}
 
 	[DllImport("user32.dll")]
 	public static extern bool SetWindowPos(IntPtr hWnd, 
 	IntPtr hWndInsertAfter, int X,int Y, int cx, int cy, uint uFlags);
 
 	[DllImport("user32.dll")]  
 	public static extern bool ShowWindow(IntPtr hWnd, int nCmdShow); 
 
 	static readonly IntPtr HWND_TOPMOST = new IntPtr(-1);
 	static readonly IntPtr HWND_NOTOPMOST = new IntPtr(-2);
 
 	const UInt32 SWP_NOSIZE = 0x0001;
 	const UInt32 SWP_NOMOVE = 0x0002;
 
 	const UInt32 TOPMOST_FLAGS = SWP_NOMOVE | SWP_NOSIZE;
 
 	public static void MakeTopMost (IntPtr fHandle)
 	{
 		SetWindowPos(fHandle, HWND_TOPMOST, 0, 0, 0, 0, TOPMOST_FLAGS);
 	}
 
 	public static void MakeNormal (IntPtr fHandle)
 	{
 		SetWindowPos(fHandle, HWND_NOTOPMOST, 0, 0, 0, 0, TOPMOST_FLAGS);
 	}
 "@
 
 
 $app = Add-Type -MemberDefinition $signature -Name Win32Window -Namespace ScriptFanatic.WinAPI -ReferencedAssemblies System.Windows.Forms -Using System.Windows.Forms -PassThru
 
 function Set-TopMost
 {
 	param(		
 		[Parameter(
 			Position=0,ValueFromPipelineByPropertyName=$true
 		)][Alias('MainWindowHandle')]$hWnd=0,
 
 		[Parameter()][switch]$Disable
 
 	)
 
 	
 	if($hWnd -ne 0)
 	{
 		if($Disable)
 		{
 			Write-Verbose "Set process handle :$hWnd to NORMAL state"
 			$null = $app::MakeNormal($hWnd)
 			return
 		}
 		
 		Write-Verbose "Set process handle :$hWnd to TOPMOST state"
 		$null = $app::MakeTopMost($hWnd)
 	}
 	else
 	{
 		Write-Verbose "$hWnd is 0"
 	}
 }
 
 
 
 function Get-WindowByTitle($WindowTitle="*")
 {
 	Write-Verbose "WindowTitle is: $WindowTitle"
 	
 	if($WindowTitle -eq "*")
 	{
 		Write-Verbose "WindowTitle is *, print all windows title"
 		Get-Process | Where-Object {$_.MainWindowTitle} | Select-Object Id,Name,MainWindowHandle,MainWindowTitle
 	}
 	else
 	{
 		Write-Verbose "WindowTitle is $WindowTitle"
 		Get-Process | Where-Object {$_.MainWindowTitle -like "*$WindowTitle*"} | Select-Object Id,Name,MainWindowHandle,MainWindowTitle
 	}
 }
 
 Exaples:
 
 gps powershell | Set-TopMost 
 
 gps powershell | Set-TopMost -disable
 
 
 Get-WindowByTitle *pad* | Set-TopMost 
 
 Get-WindowByTitle textpad | Set-TopMost -Disable
`

