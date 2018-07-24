---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 845
Published Date: 
Archived Date: 2009-02-05t17
---

# huddledtricks - 

## Description

a simple trick to show and hide windows — and an extra bonus trick ;)

## Comments



## Usage



## TODO



## class

`hide-window`

## Code

`#
 #
 ###################################################################################################
 ###################################################################################################
 add-type @"
 using System;
 using System.Runtime.InteropServices;
 public class Tricks {
    [DllImport("user32.dll")]
    private static extern bool ShowWindowAsync(IntPtr hWnd, int nCmdShow);
 
 	public static void ShowWindow(IntPtr hWnd) { ShowWindowAsync(hWnd,5); }
 	public static void HideWindow(IntPtr hWnd) { ShowWindowAsync(hWnd,0); }
 }
 "@
 
 $dictionary = [System.Collections.Generic.Dictionary``2]
 $list = [System.Collections.Generic.List``1]
 $string = [String]
 $intPtr = [IntPtr]
 
 $intPtrList = $list.AssemblyQualifiedName -replace "1,","1[[$($intPtr.AssemblyQualifiedName)]],"
 $windows = new-object ($dictionary.AssemblyQualifiedName -replace "2,","2[[$($string.AssemblyQualifiedName)],[$intPtrList]],")
 
 
 function Hide-Window {
 <#
 .SYNOPSIS
    Hide a window
 .DESCRIPTION
    Hide a window by handle or by process name. Hidden windows are gone: they don't show up in alt+tab, or on the taskbar, and they're invisible. The only way to get them back is with ShowWindow.
    
    Windows hidden by Hide-Window are pushed onto a stack so they can be retrieved by Show-Window. Since they're invisible, they're very hard to find otherwise.
 .NOTES
    See also Show-Window
 .PARAMETER WindowHandle
    The window handle to a specific window we want to hide.
 .PARAMETER Name
    The name of a running process whos windows you want to hide.
 .EXAMPLE
    Get-Process PowerShell | Hide-Window; Sleep 5; Show-Window
    
    Hides the PowerShell window(s) for 5 seconds
 .LINK
    http://HuddledMasses.org/stupid-powershell-tricks
 #>
 [CmdletBinding()]
 PARAM (
    [Parameter(Mandatory=$false, ValueFromPipelineByPropertyName=$true)]
    [Alias("MainWindowHandle","Handle")]
    [IntPtr]$WindowHandle=[IntPtr]::Zero
 ,
    [Parameter(Mandatory=$false, ValueFromPipelineByPropertyName=$true)]
    [Alias("ProcessName")]
    [String]$Name
 )
 PROCESS { 
 	(Push-Window $WindowHandle $Name) | ForEach-Object { [Tricks]::HideWindow( $_ ) }
 }
 }
 
 function Show-Window {
 <#
 .SYNOPSIS
    Show a window
 .DESCRIPTION
    Show a window by handle or by process name. If you call it without any parameters, will show all hidden windows.
 .NOTES
    See also Hide-Window
 .PARAMETER WindowHandle
    The window handle to a specific window we want to show.
 .PARAMETER Name
    The name of a running process whos windows you want to show.
 .EXAMPLE
    Get-Process PowerShell | Hide-Window; Sleep 5; Show-Window
    
    Hides the PowerShell window(s) for 5 seconds
 .LINK
    http://HuddledMasses.org/stupid-powershell-tricks
 #>
 [CmdletBinding()]
 
 PARAM (
 [Parameter(Mandatory=$false, ValueFromPipelineByPropertyName=$true)]
 [Alias("MainWindowHandle","Handle")]
 [IntPtr]$WindowHandle=[IntPtr]::Zero
 ,
 [Parameter(Mandatory=$false, ValueFromPipelineByPropertyName=$true)]
 [Alias("ProcessName")]
 [String]$Name
 )
 PROCESS { 
 	(Pop-Window $WindowHandle $Name) | ForEach-Object { [Tricks]::ShowWindow( $_ ) }
 }
 }
 
 FUNCTION Push-Window {
    [CmdletBinding()]
 	PARAM ([IntPtr]$Handle,[String]$ProcessName)
 	
 	[IntPtr[]]$Handles = @($Handle)
 	if(!$ProcessName) { $ProcessName = "--Unknown--" }
    
 	if(!$Windows.ContainsKey($ProcessName)) {
 		$windows.Add($ProcessName, (New-Object $intPtrList))
 	} 
 	if($Handle -eq [IntPtr]::Zero) {
 		[IntPtr[]]$Handles = @(Get-Process $ProcessName | % { $_.MainWindowHandle } )
 	}
 	$windows[$ProcessName].AddRange($Handles)
 	write-output $Handles
 }
 
 FUNCTION Pop-Window {
    [CmdletBinding()]
 	PARAM ([IntPtr]$Handle,[String]$ProcessName)
 	
 	[IntPtr[]]$Handles = @($Handle)
 	if(!$ProcessName) { $ProcessName = "--Unknown--" }
    
 	if(($Handle -eq [IntPtr]::Zero) -and $windows[$ProcessName] ) {
 		write-output $windows[$ProcessName]
 		$Null = $windows[$ProcessName].Clear()
 	} elseif($Handle -ne [IntPtr]::Zero){
 		$Null = $windows[$ProcessName].Remove( $Handle )
 	}
 
 	write-output $Handles
 }
 
 
 Add-Type -Assembly System.Windows.Forms
 function Wiggle-Mouse {
 <#
 .SYNOPSIS
    Wiggle the mouse until you Ctrl+Break to stop the script.
 .DESCRIPTION
    Wiggle-Mouse randomly moves the mouse by a few pixels every few milliseconds until you stop the script.
 .PARAMETER Variation
    The maximum distance to move the mosue in any direction.  Values that are too small aren't noticeable, and values that are too big make the user "loose" the mouse and they have no idea what happened.
 .PARAMETER Name
    The name of a running process whos windows you want to hide.
 .EXAMPLE
    Get-Process PowerShell | Hide-Window; Wiggle-Mouse -Duration 5;  Get-Process PowerShell | Show-Window
    
    Hides the PowerShell window and wiggle the mouse for five seconds ... :D
 .LINK
    http://HuddledMasses.org/stupid-powershell-tricks
 #>
 [CmdletBinding()]
 PARAM (
 [Parameter(Position=0)][Int]$Variation=80,
 [Parameter(Position=1)][Int]$Sleep=400,
 [Parameter(Position=2)][Int]$Duration=0
 )
    if(!$Duration) { Write-Host "Ctrl+C to stop wiggling :)" }
 	$screen = [System.Windows.Forms.SystemInformation]::VirtualScreen
 	$random = new-object Random
    $Duration *= -1000
 	While($Duration -le 0) {
 		[Windows.Forms.Cursor]::Position ="$(
 			$random.Next(	[Math]::Max( $screen.Left, ([System.Windows.Forms.Cursor]::Position.X - $Variation) ), 
 								[Math]::Min( $screen.Right, ([System.Windows.Forms.Cursor]::Position.X + $Variation) )	)
 							),$(
 			$random.Next(	[Math]::Max( $screen.Top, ([System.Windows.Forms.Cursor]::Position.Y - $Variation) ), 
 								[Math]::Min( $screen.Bottom, ([System.Windows.Forms.Cursor]::Position.Y + $Variation) )	)
 							)" 
 		sleep -milli $Sleep
       $Duration += $Sleep
 	} 
 }
 
 
 Export-ModuleMember Show-Window, Hide-Window, Wiggle-Mouse
`

