---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6482
Published Date: 2016-08-20t03
Archived Date: 2016-08-22t10
---

# install-tempfont.ps1 - 

## Description

temporarily (until restart) makes a font available without needing to install it (and thus, without need for admin rights).

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 add-type -name Session -namespace "" -member @"
 [DllImport("gdi32.dll")]
 public static extern int AddFontResource(string filePath);
 "@
 
 foreach($font in Get-ChildItem -Recurse -Include *.ttf, *.otg) { [Session]::AddFontResource($font.FullName) }
`

