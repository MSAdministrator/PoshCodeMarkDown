---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 585
Published Date: 2009-09-15t02
Archived Date: 2014-08-14t23
---

# export-screenshot - 

## Description

export-screenshot and get-screenshot let you get a system.drawing.bitmap of the full virtual screen or a cropped area of it, and save it to file …

## Comments



## Usage



## TODO



## function

`get-screenshot`

## Code

`#
 #
 #####################################################################################
 #####################################################################################
 
 
 $null = [Reflection.Assembly]::LoadWithPartialName( "System.Windows.Forms" )
 
 function Get-Screenshot ([Drawing.Rectangle]$bounds) {
    $screenshot = new-object Drawing.Bitmap $bounds.width, $bounds.height
    $graphics = [Drawing.Graphics]::FromImage($screenshot)
    $graphics.CopyFromScreen( $bounds.Location, [Drawing.Point]::Empty, $bounds.size)
 	$graphics.Dispose()
    return $screenshot
 }
 
 function Export-Screenshot {
 PARAM (
    [string]$FilePath, 
    [Drawing.Rectangle]$bounds = [Windows.Forms.SystemInformation]::VirtualScreen
 )
    write-host "FilePath: $($FilePath)" -fore green
 
    if(!(split-path $FilePath).Length) { 
       $FilePath = join-path $pwd (split-path $FilePath -leaf)
       Write-Host "FilePath: $($FilePath)" -fore magenta
    }
    
    write-host "FilePath: $($FilePath)" -fore cyan
 
    $screenshot = Get-Screenshot $bounds
    $screenshot.Save( $($FilePath) )
    $screenshot.Dispose()
    gci $FilePath
 }
`

