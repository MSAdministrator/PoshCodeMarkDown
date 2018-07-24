---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 0.25
Encoding: ascii
License: cc0
PoshCode ID: 2000
Published Date: 2011-07-21t09
Archived Date: 2016-03-05t01
---

# resize-image - 

## Description

a simple image resizer …

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 param($source = "C:\Windows\Web\Wallpaper\Windows\img0.jpg", $destination = "$Home\Pictures\thumb0.png", $scale = 0.25)
 Add-Type -Assembly PresentationCore
 
 $image = New-Object System.Windows.Media.Imaging.TransformedBitmap (New-Object System.Windows.Media.Imaging.BitmapImage $source),
                                                                    (New-Object System.Windows.Media.ScaleTransform $scale,$scale)
 [System.Windows.Clipboard]::SetImage( $image )
 
 $stream = [System.IO.File]::Open($destination, "OpenOrCreate")
 $encoder = New-Object System.Windows.Media.Imaging.$([IO.Path]::GetExtension($destination).substring(1))BitmapEncoder
 $encoder.Frames.Add([System.Windows.Media.Imaging.BitmapFrame]::Create($image))
 $encoder.Save($stream)
 $stream.Dispose()
`

