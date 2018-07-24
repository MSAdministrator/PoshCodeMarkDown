---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 87.5
Encoding: ascii
License: cc0
PoshCode ID: 1560
Published Date: 
Archived Date: 2010-01-06t03
---

# o-christmastree - 

## Description

and dropping mirror balls too. a silly little powerboots script!

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 ipmo PowerBoots
 
 
 boots { 
    $global:lights = @()
    grid { 
       rectangle -width 20 -height 40 -fill brown -HorizontalAlignment Center -VerticalAlignment Bottom
       polygon -points { "87.5,0","0,150","175,150","87.5,0" } -margin "0,25,0,40" -fill darkgreen 
       1..60 | % { ellipse -width 2 -height 3 -fill red -HorizontalAlignment Center -VerticalAlignment Top -margin { $y = get-random -min 5 -max 140; $x = get-random -min 5 -max $($y * .75 + 5); if(Get-Random 1,0){ "{0},{1},0,0" -f $x,($y+30) } else { "0,{1},{0},0" -f $x,($y+30) } } -ov +global:lights }
       polygon -points { "15,0","25,30","0,10","30,10","5,30","15,0" } -fill white -HorizontalAlignment Center -VerticalAlignment Top -ov +global:lights
    }
 } -on_sourceinitialized {
    $lights | % { $_.BeginAnimation( [System.Windows.Shapes.Shape]::OpacityProperty, 
    $(DoubleAnimation -From "1.0" -To $(Get-Random -Min 0.1 -Max 0.5) -Duration $("0:0:0.{0}" -f (Get-Random -Min 5 -Max 99)) -AutoReverse -RepeatBehavior ([system.windows.media.animation.RepeatBehavior]::Forever))) }
 } -on_mouseleftbuttondown { 
    $this.DragMove() 
 } -windowstyle none -allowstransparency -background transparent -Topmost
 
 
 
 boots {
    $global:lights = @()
    canvas {
       rectangle -width 5 -height 500 -fill Gray -HorizontalAlignment Center -VerticalAlignment Bottom -"Canvas.Left" 82.5
       ellipse -Width 50 -Height 50 -HorizontalAlignment Center -VerticalAlignment Top -Fill White -OV +global:lights -"Canvas.Left" 60
       polygon -HorizontalAlignment Center -VerticalAlignment Top -points "0,0", "75,10",   "80,14",
                                                                          "0,0", "-80,-10", "-75,-14",
                                                                          "0,0", "60,40",   "65,42",
                                                                          "0,0", "-50,-30", "-57,-32",
                                                                          "0,0", "75,-10",  "85,-16",
                                                                          "0,0", "-60,40",  "-65,42" -fill LightYellow -margin "85,25,0,0" -Opacity 0.3 -OV +global:lights
    } -width 170 -height 500
 } -on_mouseleftbuttondown { 
    $this.DragMove() 
 } -on_sourceinitialized {
    $lights | % {  
       $_.BeginAnimation( [System.Windows.Controls.Canvas]::TopProperty, $(DoubleAnimation -From 50 -To 550 -Duration '0:0:10') )
    } 
    @($lights)[-1].BeginAnimation( [System.Windows.Shapes.Shape]::OpacityProperty, 
       $(DoubleAnimation -From "0.3" -To "0.0" -Duration "0:0:1" -AutoReverse -RepeatBehavior ([system.windows.media.animation.RepeatBehavior]::Forever)))
 } -windowstyle none -allowstransparency -background transparent
`

