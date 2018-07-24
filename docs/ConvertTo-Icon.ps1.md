---
Author: chad miller
Publisher: 
Copyright: 
Email: 
Version: 1.0
Encoding: ascii
License: cc0
PoshCode ID: 2765
Published Date: 2011-07-03t13
Archived Date: 2016-09-07t05
---

# convertto-icon - 

## Description

converts image files to icon files

## Comments



## Usage



## TODO



## function

`convertto-icon`

## Code

`#
 #
 [Reflection.Assembly]::LoadWithPartialName("System.Drawing") | Out-Null
 
 
 
 function ConvertTo-Icon
 {
     [cmdletbinding()]
     param([Parameter(Mandatory=$true, ValueFromPipeline = $true)] $Path)
     
     process{
         if ($Path -is [string])
         { $Path = get-childitem $Path }
            
         $Path | foreach {
             $image = [System.Drawing.Image]::FromFile($($_.FullName))
 
             $FilePath =  "{0}\{1}.ico" -f $($_.DirectoryName), $($_.BaseName)
             $stream = [System.IO.File]::OpenWrite($FilePath)
 
             $bitmap = new-object System.Drawing.Bitmap $image
             $bitmap.SetResolution(72,72)
             $icon = [System.Drawing.Icon]::FromHandle($bitmap.GetHicon())
             $icon.Save($stream)
             $stream.Close()
         }
     }
       
  }
`

