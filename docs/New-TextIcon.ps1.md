---
Author: chrissylemaire
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6013
Published Date: 2016-09-14t15
Archived Date: 2016-05-17t13
---

# new-texticon - 

## Description

new-texticon allows you to create an icon made of text. this was useful for me when creating a notifyicon that contained the used percentage of memory on a vm host. output to actual file optional using $path.

## Comments



## Usage



## TODO



## function

`new-texticon`

## Code

`#
 #
 
 Function New-TextIcon {
     [CmdletBinding()] 
 	Param(
         	[Parameter(Mandatory=$true)] 
         	[string]$Text,
         	[string]$Path,
         	[int]$Height = 16,
         	[int]$Width = 16,
 		[int]$FontSize = 9
 	)
    
     DynamicParam  {
 		[void][System.Reflection.Assembly]::LoadWithPartialName("System.windows.forms")
 		$fontlist = (New-Object System.Drawing.Text.InstalledFontCollection).Families
 		$stylelist = [System.Enum]::GetNames([System.Drawing.FontStyle])
 		$colorlist = [System.Enum]::GetNames([System.Drawing.KnownColor])
 		
 		$attributes = New-Object System.Management.Automation.ParameterAttribute
 		$attributes.ParameterSetName = "__AllParameterSets"
 		$attributes.Mandatory = $false
     
 		$validationset = New-Object -Type System.Management.Automation.ValidateSetAttribute -ArgumentList $fontlist
 		$collection = New-Object -Type System.Collections.ObjectModel.Collection[System.Attribute]
 		$collection.Add($attributes)
 		$collection.Add($validationset)
 		$fontname = New-Object -Type System.Management.Automation.RuntimeDefinedParameter("FontName", [String], $collection)
         $fontname.Value = "Helvetica"
 
 		$validationset = New-Object -Type System.Management.Automation.ValidateSetAttribute -ArgumentList $stylelist
 		$collection = New-Object -Type System.Collections.ObjectModel.Collection[System.Attribute]
 		$collection.Add($attributes)
 		$collection.Add($validationset)
 		$fontstyle = New-Object -Type System.Management.Automation.RuntimeDefinedParameter("FontStyle", [String], $collection)
         $fontstyle.Value = "Regular"
  
 		$validationset = New-Object -Type System.Management.Automation.ValidateSetAttribute -ArgumentList $colorlist
 		$collection = New-Object -Type System.Collections.ObjectModel.Collection[System.Attribute]
 		$collection.Add($attributes)
 		$collection.Add($validationset)
 		$background = New-Object -Type System.Management.Automation.RuntimeDefinedParameter("Background", [String], $collection)
         $background.Value = "Transparent"
             
 		$validationset = New-Object -Type System.Management.Automation.ValidateSetAttribute -ArgumentList $colorlist
 		$collection = New-Object -Type System.Collections.ObjectModel.Collection[System.Attribute]
 		$collection.Add($attributes)
 		$collection.Add($validationset)
 		$foreground = New-Object -Type System.Management.Automation.RuntimeDefinedParameter("Foreground", [String], $collection)     
 		$foreground.Value = "White"
             
 		$newparams = New-Object System.Management.Automation.RuntimeDefinedParameterDictionary
 		$newparams.Add("FontName", $fontname)
 		$newparams.Add("FontStyle", $fontstyle)
 		$newparams.Add("Background", $background)
 		$newparams.Add("Foreground", $foreground)
 	
 	return $newparams
 	}
 
     PROCESS {
         
         $fontname = $fontname.value; $fontstyle = $fontstyle.value; 
         $background = $background.value; $foreground = $foreground.value
   
         $fontstyle = [System.Drawing.FontStyle]::$fontstyle
         $background = [System.Drawing.Brushes]::$background
         $foreground = [System.Drawing.Brushes]::$foreground
         
         $bmp = New-Object System.Drawing.Bitmap $width,$height
         $font = New-Object System.Drawing.Font($fontname,$fontsize,$fontstyle)
         $graphics = [System.Drawing.Graphics]::FromImage($bmp) 
         $graphics.FillRectangle($background,0,0,$wsWidth,$height)
         
         $graphics.DrawString($text,$font,$foreground,0,0)
         $graphics.Dispose()
         if ($Path.Length -gt 0) {
             $ext = ([System.IO.Path]::GetExtension($Path)).TrimStart(".")
             $null = $bmp.Save($path, $ext) 
         }
         $icon = [System.Drawing.Icon]::FromHandle($bmp.GetHicon())
         return $icon
     }
 }
 
 $notify= New-Object System.Windows.Forms.NotifyIcon 
 $notify.Icon =  $icon
 
 Invoke-Item C:\temp\test.png
`

