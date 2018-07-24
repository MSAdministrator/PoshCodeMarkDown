---
Author: bielawb
Publisher: 
Copyright: 
Email: 
Version: 0.5
Encoding: ascii
License: cc0
PoshCode ID: 5730
Published Date: 2015-02-06t13
Archived Date: 2015-02-13t04
---

# wpf v3 demo - 

## Description

demo of wpf with powershell v3 – using [class]@{ property = value }, flexible syntax, simplified where. also using type accelerators. not really smartest way to create wpf (no xaml – as bad as it sounds). so don’t treat this as very good example® or wpf in use. ;)

## Comments



## Usage



## TODO



## class

``

## Code

`#
 #
 
 $Akceleratory = 
     [PSObject].
     Assembly.
     GetType("System.Management.Automation.TypeAccelerators")
 
 Add-Type -AssemblyName PresentationCore, PresentationFramework -PassThru |
     Where-Object IsPublic |
     ForEach-Object {
         $Class = $_
         try {
             $Akceleratory::Add($Class.Name,$Class)
         } catch {
             "Failed to add $($Class.Name) accelerator pointing to $($Class.FullName)"
         }
     }
 
 [Window]@{
     OpacityMask = [DrawingBrush]@{
         Drawing = [DrawingGroup]@{
             Children = & {
                 $Kolekcja = New-Object DrawingCollection 
                 $Kolekcja.Add([GeometryDrawing]@{
                     Brush = 'Black'
                     Geometry = [EllipseGeometry]@{
                         radiusX = 0.48
                         radiusY = 0.48
                         Center = '0.5,0.5'
                     }
                 })
                 $Kolekcja.Add([GeometryDrawing]@{
                     Brush = 'Transparent'
                     Geometry = [RectangleGeometry]@{
                         Rect = '0,0,1,1'
                     }
                 })
                 , $Kolekcja
             }
         }
     }
     Background = [LinearGradientBrush]@{
         Opacity = 0.5
         StartPoint = '0,0.5'
         Endpoint = '1,0.5'
         GradientStops = & {
             $Stopki = New-Object GradientStopCollection
             $Colors = 'Blue', 'Green'
                 foreach ($i in 0..1) {
                 $Stopki.Add(
                     [GradientStop]@{
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
     Effect = [DropShadowEffect]@{
         BlurRadius = 10
     }
     TopMost = $true
     Content = & {
         $Stos = [StackPanel]@{
             VerticalAlignment = 'Center'
             HorizontalAlignment = 'Center'
         }
 
         $Stos.AddChild(
             [Label]@{
                 Content = 'PowerShell Rocks!'
                 FontSize = 80
                 FontFamily = 'Consolas'
                 Foreground = 'White'
                 Effect = [DropShadowEffect]@{
                     BlurRadius = 5
                 }
             }
         )
         , $Stos
     }
 } | ForEach-Object {
     $_.Add_MouseLeftButtonDown({
         $this.DragMove()
     })
     $_.Add_MouseRightButtonDown({
         $this.Close()
     })
     $_.ShowDialog() | Out-Null
 }
`

