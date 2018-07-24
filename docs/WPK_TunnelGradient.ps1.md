---
Author: bartekb
Publisher: 
Copyright: 
Email: 
Version: 0.5
Encoding: ascii
License: cc0
PoshCode ID: 2664
Published Date: 2011-05-08t06
Archived Date: 2011-05-11t01
---

# wpk_tunnelgradient - 

## Description

i noticed that wpf has lineargradient and radialgradient. wanted something more… square in size. first tried to use something that allowed me to build brush easy, but it was hard to change it into something portable. so i just used new-object and now you can use it in wpk, powerboots or show-ui, whichever you prefer. ;) this script is building actual function and shows some demo usage + load required assemblies and makes sure we are in sta (some controls barfed at me when i tried to leave it -mta). i only hope it’s not done already… ;) corners are rounded, so that it looks more soft.

## Comments



## Usage



## TODO



## script

`new-tunnelgradientbrush`

## Code

`#
 #
 [CmdletBinding()]
 param (
     [string]$InsideColor = 'White',
     [string]$OutsideColor = 'Transparent',
     [int]$BorderThickness = 5
 )
 
 Add-Type -AssemblyName PresentationCore
 Add-Type -AssemblyName PresentationFramework
 Add-Type -AssemblyName WindowsBase
 
 if ([Threading.Thread]::CurrentThread.GetApartmentState() -eq 'MTA') {
     Write-Warning @'
 This script is using controls, that require STA mode. You can:
 * Try to run 'PowerShell -STA' and launch the script again
 * Run it from PowerShell ISE where STA is default mode.
 See you than! ;)
 '@
     exit
 }
 
 Function New-TunnelGradientBrush {
 [CmdletBinding()]
 param (
     [Parameter(Mandatory = $true, HelpMessage = 'Color that will fill interior of the brush')]
     [System.Windows.Media.Color]$InsideColor,
     [Parameter(Mandatory = $true, HelpMessage = 'Color that will be used for border of the brush')]
     [System.Windows.Media.Color]$OutsideColor,
     [Parameter(Mandatory = $true, HelpMessage = 'Size of border as part of width, acceptable values: 0 - 0.49')]
     [ValidateRange(0,49)]
     [int]$BorderThickness
 )
     Set-StrictMode -Version Latest
     
     $Left = New-Object Windows.Point 0, 0.5
     $Right = New-Object Windows.Point 1, 0.5
     $Top = New-Object Windows.Point 0.5, 0
     $Bottom = New-Object Windows.Point 0.5, 1
 
     $LeftTop = New-Object Windows.Point 0, 0
     $LeftBottom = New-Object Windows.Point 0, 1
     $RightTop = New-Object Windows.Point 1, 0
     $RightBottom = New-Object Windows.Point 1, 1
     
     
     [double]$StartOffset = $BorderThickness / 100
     [double]$EndOffset = - ($BorderThickness / 100) + 1
     $NarrowSize = 100 - 2 * $BorderThickness
     $CornerSize = $BorderThickness
  
     
     $GradientStops = New-Object Windows.Media.GradientStopCollection
     @{ 0 = $OutsideColor
         $StartOffset = $InsideColor
         $EndOffset = $InsideColor
         1 = $OutsideColor
     }.GetEnumerator() | Foreach { $GradientStops.Add($(New-Object Windows.Media.GradientStop $_.Value, $_.Name)) }
 
     $GradientStopsRadial = New-Object Windows.Media.GradientStopCollection
     @{ 1 = $OutsideColor
        0 = $InsideColor
     }.GetEnumerator() | Foreach { $GradientStopsRadial.Add($(New-Object Windows.Media.GradientStop $_.Value, $_.Name)) }
     
     
     $BrushLeftRight = New-Object Windows.Media.LinearGradientBrush -Property @{ 
         StartPoint = $Left 
         EndPoint = $Right 
         GradientStops = $GradientStops
     }
 
     $BrushTopBottom = New-Object Windows.Media.LinearGradientBrush -Property @{ 
         StartPoint = $Top
         EndPoint = $Bottom 
         GradientStops = $GradientStops
     }
     
     $CommonProperty = @{
         RadiusX = 1 
         RadiusY = 1 
         GradientStops = $GradientStopsRadial
     }
 
     $BrushLeftTop = New-Object Windows.Media.RadialGradientBrush -Property (@{ 
         Center = $RightBottom 
         GradientOrigin = $RightBottom 
     } + $CommonProperty)
 
     $BrushLeftBottom = New-Object Windows.Media.RadialGradientBrush -Property (@{ 
         Center = $RightTop 
         GradientOrigin = $RightTop 
     } + $CommonProperty)
         
     $BrushRightTop = New-Object Windows.Media.RadialGradientBrush -Property (@{ 
         Center = $LeftBottom 
         GradientOrigin = $LeftBottom
     } + $CommonProperty)
 
     $BrushRightBottom = New-Object Windows.Media.RadialGradientBrush -Property (@{ 
         Center = $LeftTop 
         GradientOrigin = $LeftTop
     } + $CommonProperty)
     
 
     $Horizontal = New-Object Windows.Controls.StackPanel -Property @{
         Background = $BrushLeftRight 
         Width = 100 
         Height = $NarrowSize
     }
     
     $Vertical = New-Object Windows.Controls.StackPanel -Property @{
         Background = $BrushTopBottom 
         Width = $NarrowSize 
         Height = 100
     }
     
     $Corners = @{
         Width = $CornerSize 
         Height = $CornerSize
     }
     
     $TopLeft = New-Object Windows.Controls.StackPanel -Property (@{
         Background = $BrushLeftTop 
         VerticalAlignment = 'Top' 
         HorizontalAlignment = 'Left'
     } + $Corners)
 
     $BottomLeft = New-Object Windows.Controls.StackPanel -Property (@{
         Background = $BrushLeftBottom 
         VerticalAlignment = 'Bottom' 
         HorizontalAlignment = 'Left'
     } + $Corners)
 
     $TopRight = New-Object Windows.Controls.StackPanel -Property (@{
         Background = $BrushRightTop 
         VerticalAlignment = 'Top' 
         HorizontalAlignment = 'Right'
     } + $Corners)
 
     $BottomRight = New-Object Windows.Controls.StackPanel -Property (@{
         Background = $BrushRightBottom 
         VerticalAlignment = 'Bottom' 
         HorizontalAlignment = 'Right'
     } + $Corners)
     
 
     $Grid = New-Object Windows.Controls.Grid
     foreach ($Control in $Vertical, $Horizontal, $BottomLeft, $BottomRight, $TopLeft, $TopRight) {
         $Grid.Children.Add($Control) | Out-Null
     }
 
     New-Object System.Windows.Media.VisualBrush $Grid    
 }
 
 try {
     $Brush = New-TunnelGradientBrush -InsideColor $InsideColor -OutsideColor $OutsideColor -BorderThickness $BorderThickness
 } catch {
     Write-Host "Try some valid values. -Verbose for more."
     Write-Verbose $_
     exit
 }
 
 $Window = New-Object Windows.Window -Property @{
     BackGround = $Brush
     Width = 200
     Height = 200
     WindowStyle = 'None'
     AllowsTransparency = $True
 }
 
 $Window.Add_MouseLeftButtonDown({ $this.DragMove() })
 $Window.Add_MouseRightButtonDown({ $this.Close() })
 $Window.ShowDialog() | Out-Null
`

