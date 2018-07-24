---
Author: james gentile
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5982
Published Date: 2016-08-20t21
Archived Date: 2016-05-17t12
---

# slideshow&nomonitorsleep; - 

## Description

this will display a slideshow and prevent the monitor from sleeping (which may be useful for other purposes). change the $folder variable to where pictures are located, and change $wait to the time between picture changes. you can move it to another monitor using windows hotkeys (win+shift+arrow). the setthreadexecutionstate p/invoke function with a parameter of (2) causes the display sleep timer to reset, so must be called in a loop to keep the screen active.

## Comments



## Usage



## TODO



## function

`new-pinvoke`

## Code

`#
 #
 $folder = "$env:userprofile\pictures"
 $wait = 10
 
 function New-PInvoke
 {
   <#
 
     .Synopsis
       Generate a powershell function alias to a Win32|C api function
     .Description
     .Example
       New-PInvoke user32 "void FlashWindow(IntPtr hwnd, bool bInvert)"
 
 Generates a function for FlashWindow which ignores the boolean return value, and allows you to make a window flash to get the user's attention. Once you've created this function, you could use this line to make a PowerShell window flash at the end of a long-running script:
         C:\PS>FlashWindow (ps -id $pid).MainWindowHandle $true
     .Parameter Library
         A C Library containing code to invoke
     .Parameter Signature
     .Parameter OutputText
         If Set, retuns the source code for the pinvoke method.
         If not, compiles the type. 
     #>
     param(
     [Parameter(Mandatory=$true, 
         HelpMessage="The C Library Containing the Function, i.e. User32")]
     [String]
     $Library,
 
     [Parameter(Mandatory=$true,
         HelpMessage="The Signature of the Method, i.e.: int GetSystemMetrics(uint Metric)")]
     [String]
     $Signature,
 
     [Switch]
     $OutputText
     )
 
     process {
         if ($Library -notlike "*.dll*") {
             $Library+=".dll"
         }
         if ($signature -notlike "*;") {
             $Signature+=";"
         }
         if ($signature -notlike "public static extern*") {
             $signature = "public static extern $signature"
         }
         $name = $($signature -replace "^.*?\s(\w+)\(.*$",'$1')
 
         $MemberDefinition = "[DllImport(`"$Library`")]`n$Signature"
 
         if (-not $OutputText) {
             $type = Add-Type -PassThru -Name "PInvoke$(Get-Random)" -MemberDefinition $MemberDefinition
             del -ea 0 Function:Global:$name
             iex "New-Item Function:Global:$name -Value { [$($type.FullName)]::$name.Invoke( `$args ) }"
         } else {
             $MemberDefinition
         }
     }
 }
 
 new-pinvoke kernel32 "int SetThreadExecutionState(uint state)"
 
 [void][reflection.assembly]::LoadWithPartialName("System.Windows.Forms")
 $form = new-object Windows.Forms.Form
 $form.Text = "Image Viewer"
 $form.WindowState= "Maximized"
 $form.controlbox = $false
 $form.formborderstyle = "0"
 $form.BackColor = [System.Drawing.Color]::black
 $pictureBox = new-object Windows.Forms.PictureBox
 $pictureBox.dock = "fill"
 $pictureBox.sizemode = 4
 $form.controls.add($pictureBox)
 $form.Add_Shown( { $form.Activate()} )
 $form.Show()
 $list=@()
 do
 {
 	if ($list.count -eq 0) 
     	{ 
 		[collections.arraylist]$list=@(dir -literalpath $folder * -include *.jpg, *.jpeg, *.bmp, *.png -recurse)
 		if ($list.count -eq 0) {"No pictures. Exiting.";exit}
     	}
     	$fileindex = get-random $list.count	
 	$file = (get-item -ea 0 $list[$fileindex].fullname)
 	$list.removeat($fileindex)
 	if ($file -eq $null -or $file.fullname -eq "") {continue}
 	
 	"$($file.fullname)"
 	
         $pictureBox.Image = [System.Drawing.Image]::Fromfile($file.fullname)
 	
 	$form.bringtofront()
 	$form.refresh()
         Start-Sleep -Seconds $wait  
 	SetThreadExecutionState(2) | out-null
 }
 While ($true)
`

