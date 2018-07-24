---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 3825
Published Date: 2013-12-14t03
Archived Date: 2016-06-03t07
---

# get-tree - 

## Description

it just needs presents under it…

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 #.Synopsis
 #.Description
 #.Parameter Trim
 #.Example
 #.Example
 #
 param(
    [switch]$Trim=$true
 , 
    [ValidateSet("Red","Blue","Cyan","Yellow","Green","Gray","Magenta","All")]
    [Parameter(Position=0)]
    [String[]]$LightColor = @("Red")
 )
 if($LightColor -contains "All") {
    $LightColor = "Red","Yellow","Green","Gray","Magenta","Cyan","Blue"
 }
 
 Clear-Host
 $OFS = "`n"
 $center = [Math]::Min( $Host.UI.RawUI.WindowSize.Width, $Host.UI.RawUI.WindowSize.Height ) - 10
 
 $Sparkle = [string][char]0x0489  
 $DkShade = [string][char]0x2593
 $Needles  = [string][char]0x0416
 
 $Width = 2
 [string[]]$Tree = $(
    "$(" " * $Center) "
    "$(" " * $Center)$([char]0x039B)"
    "$(" " * ($Center - 1))$($Needles * 3)"
   
    for($i = 3; $i -lt $center; $i++) {
       (" " * ($Center - $i)) + (Get-Random $Needles, " ") + ($Needles * (($Width * 2) + 1)) + (Get-Random $Needles, " ")
       $Width++
    }
    for($i = 0; $i -lt 4; $i++) {
       " " * ($Center + 2)
    }
 ) 
 
 $TreeOn = $Host.UI.RawUI.NewBufferCellArray( $Tree, "DarkGreen", "DarkMagenta" )
 $TreeOff = $Host.UI.RawUI.NewBufferCellArray( $Tree, "DarkGreen", "DarkMagenta" )
 
 for($x=-2;$x -le 2;$x++) { 
    for($y=0;$y -lt 4;$y++) {
       $TreeOn[($center+$y),($center+$x)] = $TreeOff[($center+$y),($center+$x)] = 
          New-Object System.Management.Automation.Host.BufferCell $DkShade, "Black", "darkMagenta", "Complete"
    }  
 }
 
 if($trim) {
 $ChanceOfLight = 50
 $LightIndex = 0
 for($y=0;$y -le $TreeOn.GetUpperBound(0);$y++) {
    for($x=0;$x -le $TreeOn.GetUpperBound(1);$x++) {
       if($TreeOn[$y,$x].Character -eq $Needles) {
          $LightIndex += 1
          if($LightIndex -ge $LightColor.Count) {
             $LightIndex = 0
          }
          if($ChanceOfLight -gt (Get-Random -Max 100)) {
             $Light = $LightColor[$LightIndex]
             $TreeOn[$y,$x] = New-Object System.Management.Automation.Host.BufferCell $Sparkle, $Light, "darkMagenta", "Complete"
             $TreeOff[$y,$x] = New-Object System.Management.Automation.Host.BufferCell $Sparkle, "Dark$Light", "darkMagenta", "Complete"
          } else { 
             $ChanceOfLight += 3
          }
       }
    }
 }
 $TreeOn[0,$Center] = $TreeOff[0,$Center] = New-Object System.Management.Automation.Host.BufferCell $Sparkle, "Yellow", "darkMagenta", "Complete"
 }
 
 
 $Coord = New-Object System.Management.Automation.Host.Coordinates (($Host.UI.RawUI.WindowSize.Width - ($Center*2))/2), 2
 $Host.UI.RawUI.SetBufferContents( $Coord, $TreeOff )
 
    sleep -milli 500
    $Host.UI.RawUI.SetBufferContents( $Coord, $TreeOn )
    sleep -milli 500
    $Host.UI.RawUI.SetBufferContents( $Coord, $TreeOff )
 }
`

