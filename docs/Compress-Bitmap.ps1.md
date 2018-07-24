---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 1290
Published Date: 2010-08-24t14
Archived Date: 2013-05-22t18
---

# compress-bitmap - 

## Description

compress a bitmap to a certain filesize, using the pscx import-bitmap, resize-bitmap, and export-bitmap … and a little trial and error.

## Comments



## Usage



## TODO



## function

`compress-bitmap`

## Code

`#
 #
 function Compress-Bitmap {
 PARAM(
    [Parameter(Mandatory=$true, ValueFromPipeline=$true)]
    [IO.FileInfo]$SourceFile
 ,
    [Parameter(Mandatory=$true, Position=1)]
    [String]$DestinationFile
 ,   
    [Parameter(Mandatory=$false)]
    [Int]$Width
 ,  [Parameter(Mandatory=$false)]
    [Int]$Height   
 ,  [Parameter(Mandatory=$false)]
    [Int]$MaxFilesize
 ,  [Parameter(Mandatory=$false)]
    [Int]$Quality = 100
 )
 BEGIN { if($SourceFile) { $SourceFile = Get-ChildItem $SourceFile } }
 PROCESS {
    [string]$intermediate = [IO.path]::GetRandomFileName() + ".jpeg"
    $bitmap = Import-Bitmap $SourceFile
    
    if($Width -and $Height) {
       $bitmap = Resize-Bitmap -Bitmap $bitmap -Width $Width -Height $Height
       $bitmap = Resize-Bitmap -Bitmap $bitmap -Percent 100
    }
    
    do { 
       Export-Bitmap -Bitmap $bitmap -Path $intermediate -Quality ($Quality--)
    } while( $MaxFilesize -and ((Get-ChildItem $intermediate).Length -gt $MaxFilesize))
    Write-Host "Output Quality: $($Quality + 1)%" -Foreground Yellow
    Move-Item $intermediate $DestinationFile -Force -Passthru
 }
 }
`

