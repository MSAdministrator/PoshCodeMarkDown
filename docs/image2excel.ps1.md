---
Author: axcoder
Publisher: 
Copyright: 
Email: 
Version: 2.5
Encoding: ascii
License: cc0
PoshCode ID: 4047
Published Date: 2013-03-27t12
Archived Date: 2013-03-30t05
---

# image2excel - 

## Description

just for fun script for converting image files to excel. very slow,  try 50×50 images first

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 <#
     .Description
     image2excel converts image to excel file
 #>
 param (
     [parameter(Mandatory=$true,
                ValueFromPipeline=$true,
                HelpMessage="Image file path"               
                )]
     [ValidateScript({Test-Path $_})]
     [String]
     $filename
 )
 [System.Reflection.Assembly]::LoadWithPartialName("System.Drawing") | Out-Null
 function color2excel($color)
 {
     $color.R * 256*256 + $color.G * 256 + $color.B
 }
 $image = New-Object System.Drawing.Bitmap $filename
 $o = New-Object -ComObject Excel.Application
 
 $wb = $o.Workbooks.Add()
 $sh = $wb.ActiveSheet
 $sh.Cells.ColumnWidth =0.25;
 $sh.Cells.RowHeight = 2.5;
 $o.Visible = $true
 $total = $image.Width * $image.Height
 foreach($x in 0..($image.Width-1))
 {
     foreach($y in 0..($image.Height-1))
     {
         Write-Progress "Exporting..." "$x,$y" -PercentComplete  ((($x * $image.Height + $y) / $total ) * 100)
         $sh.Cells.Item($y+1, $x+1).Interior.Color = color2excel($image.GetPixel($x, $y))
     }
 }
`

