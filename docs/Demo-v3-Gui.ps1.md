---
Author: bielawb
Publisher: 
Copyright: 
Email: 
Version: 0.5
Encoding: ascii
License: cc0
PoshCode ID: 3900
Published Date: 2013-01-14t09
Archived Date: 2013-04-29t03
---

# demo-v3-gui.ps1 - 

## Description

very simple, basic script to show how you can utilize v3 syntax to build guis with very little code…

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 
 [Windows.Window]@{
     OpacityMask = [Windows.Media.DrawingBrush]@{
         Drawing = [Windows.Media.GeometryDrawing]@{
             Brush = 'Black'
             Geometry = [Windows.Media.EllipseGeometry]@{
                 radiusX = 123
                 radiusY = 321
             }
         }
     }
     Background = [Windows.Media.LinearGradientBrush]@{
         Opacity = 0.5
         StartPoint = '0,0.5'
         Endpoint = '1,0.5'
         GradientStops = & {
             $Stopki = New-Object Windows.Media.GradientStopCollection
             $Colors = 'Blue', 'Green'
                 foreach ($i in 0..1) {
                 $Stopki.Add(
                     [Windows.Media.GradientStop]@{
                         Color = $Colors[$i]
                         Offset = $i
                     }
                 )
             }
             , $Stopki
         }            
     }
     Width = 800
     Height = 400
     WindowStyle = 'None'
     AllowsTransparency = $true
     Effect = [Windows.Media.Effects.DropShadowEffect]@{
         BlurRadius = 10
     }
     Content = & {
         $Stos = [Windows.Controls.StackPanel]@{
             VerticalAlignment = 'Center'
             HorizontalAlignment = 'Center'
         }
 
         $Stos.AddChild(
             [Windows.Controls.Label]@{
                 Content = 'PowerShell Rocks!'
                 FontSize = 80
                 FontFamily = 'Consolas'
                 Foreground = 'White'
                 Effect = [Windows.Media.Effects.DropShadowEffect]@{
                     BlurRadius = 5
                 }
             }
         )
         , $Stos
     }
 } | foreach {
     $_.Add_MouseLeftButtonDown({
         $this.DragMove()
     })
     $_.Add_MouseRightButtonDown({
         $this.Close()
     })
     $_.ShowDialog()
 }
`

