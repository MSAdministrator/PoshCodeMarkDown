---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 1.0
Encoding: ascii
License: cc0
PoshCode ID: 617
Published Date: 2009-09-30t19
Archived Date: 2017-05-16t03
---

# get-imagemetadata - 

## Description

get-imagemetadata lets you access the exif, xmp and other metadata about image files … this should only be taken as an example and a reference, not for solid production work — in other words, i won’t take the blame if you zero out all your jpgs trying to modify this to readwrite instead of just read

## Comments



## Usage



## TODO



## function

`get-imagemetadata`

## Code

`#
 #
 #####################################################################################################
 #####################################################################################################
 PARAM($file)
 BEGIN {
    $null = [Reflection.Assembly]::LoadWithPartialName("PresentationCore");
    
    function Get-ImageMetadata {
       PARAM([System.Windows.Media.Imaging.BitmapFrame]$bitmapFrame, [string]$path)
       PROCESS {
          if($path -is [string]) {
             $next=$bitmapFrame.MetaData.GetQuery($path);
             if($next.Location){
                $next | ForEach-Object { 
                   Get-ImageMetadata $bitmapFrame "$($next.Location)$_" 
                }
             } else {
                if($path.Split("/")[-1] -match "{ushort=(?<code>\d+)}") {
                   $path = [int]$matches["code"]
                }
                Add-Member -in ($Global:ImageMetaData) -Type NoteProperty -Name $path -value $next -Force
             }
          } else {
             $bitmapFrame.Metadata | ForEach-Object { Get-ImageMetadata $bitmapFrame $_ }
          }
       }
    }
 }
 PROCESS {
    if($_) { $file = $_ }
    
    if($file -is [IO.FileInfo]) {
       $file = [string]$file.FullName;
    } elseif($file -is [String]) {
       $file = [string](Resolve-Path $file)
    } 
 
    $Global:ImageMetaData = New-Object IO.FileInfo $file
    
    $stream = new-object IO.FileStream $file, ([IO.FileMode]::Open), ([IO.FileAccess]::Read), ([IO.FileShare]::Read);
    & {
       $decoder = [System.Windows.Media.Imaging.BitmapDecoder]::Create( $stream, "None", "Default" )
       $bitmapFrame = $decoder.Frames[0];
       $bitmapFrame.Metadata | ForEach-Object { Get-ImageMetadata $bitmapFrame $_ }
    }
    trap { 
       Write-Error "WARNING: $_"
       continue; 
    }
    $stream.Close()
    $stream.Dispose()
    
    Write-Output $Global:ImageMetaData
 }
`

