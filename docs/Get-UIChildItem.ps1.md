---
Author: bartekb
Publisher: 
Copyright: 
Email: 
Version: 0.5
Encoding: ascii
License: cc0
PoshCode ID: 2673
Published Date: 2011-05-12t15
Archived Date: 2017-05-16t03
---

# get-uichilditem - 

## Description

very simple script that will create file list from -path in simple ui generated using show-ui. tooltip show size and last write time of a file, and once clicked – it will pass fullname down the pipe. it was created mainly as a attempt to use some animated effects in show-ui.

## Comments



## Usage



## TODO



## module

``

## Code

`#
 #
 
 [CmdletBinding()]
 param($Path)
 
 <#
     Quick check to see if we have necessary module...
 #>
 
 if (!(Get-Module ShowUI)) {
     try {
         Import-Module ShowUI
     } catch {
         Write-Warning "Can't load ShowUI module. I quit! For more details about error use -Verbose parameter."
         Write-Verbose $_
     }
 }   
 
 <#
     Show UI builds UI structure...
 #>
 
 Show-UI  -Parameters $Path {
     param ($Path)
     DockPanel -Height 400 {
         ScrollViewer {
             ItemsControl {
                 <#
                     Hash that will set common button's options and some effects...
                 #>
             
                 $Effect = @{
                     Effect = {
                         New-Object Windows.Media.Effects.BlurEffect -Property @{
                             Radius = 3
                         }
                     }
                     On_MouseEnter = {
                         $Clear = DoubleAnimation -From 3 -To 1 -Duration '0:0:0.5'
                         $DependencyRadius = $This.Effect.GetType()::RadiusProperty
                         $This.Effect.BeginAnimation($DependencyRadius,$Clear)
                     }
                     On_MouseLeave = {
                         $Blur = DoubleAnimation -From 1 -To 3 -Duration '0:0:0.5'
                         $DependencyRadius = $This.Effect.GetType()::RadiusProperty
                         $This.Effect.BeginAnimation($DependencyRadius,$Blur)
                     }
                 }
                 
                 <#
                     Time to read our folder. We ignore folders, only files will be listed.
                     As you can see it's pretty easy to pass $Path parameter to UI, and it's not global... :)
                 #>
                 
                 foreach ($File in Get-ChildItem $Path | where { !$_.PSIsContainer } ) {
                     Button @Effect -Content $File.Name -On_Click ([scriptblock]::Create(@"
                         Write-UIOutput $($File.FullName)
                         `$ShowUI.ActiveWindow.Close()
 "@)) -ToolTip "File size: $($File.Length) bytes, Last modified: $($File.LastWriteTime.ToShortDateString())"
                     
                 }
`

