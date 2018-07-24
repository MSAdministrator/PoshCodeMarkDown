---
Author: oleg medvedev
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 621
Published Date: 2009-10-01t13
Archived Date: 2017-04-02t18
---

# resizer of pictures - 

## Description

script for total resize of all pictures in one or multiple directories.

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 [reflection.assembly]::LoadWithPartialName("System.Drawing")
 
 
   $toresize=@("n:\path1", "n:\path2\", "s:\path3")
 }
 
 #$null = $Host.UI.RawUI.ReadKey("NoEcho,IncludeKeyDown")
 
 $error.clear()
 
 
 Get-ChildItem -recurse $toresize -include *.jpg  | foreach {
     $error | out-file $logfile -append -encoding default
     $error[($error.count-1)].TargetObject | out-file $logfile -append -encoding default
     echo $_>>$logfile
     $error.clear()
   }
   if ($OldBitmap.Width -lt $OldBitmap.Height) { $LongSide=$OldBitmap.Height }
       $newH=$SizeLimit
       $newW=[int]($OldBitmap.Width*$SizeLimit/$OldBitmap.Height)
     } else {
       $newW=$SizeLimit
       $newH=[int]($OldBitmap.Height*$newW/$OldBitmap.Width)
     }
     $g=[System.Drawing.Graphics]::FromImage($NewBitmap)
 
     $OldBitmap.Dispose()
     $n=get-childitem $name
       if ($n.Exists -and $_.Exists) {
         $name=$_.FullName
       }
     }
     
     Write-host -NoNewLine "."
     $OldBitmap.Dispose()
   }
 }
 
 
 
 Get-ChildItem -recurse $toresize -include *.bmp, *.tif, *.gif | foreach {
     $error | out-file $logfile -append -encoding default
     $error[($error.count-1)].TargetObject | out-file $logfile -append -encoding default
     $error.clear()
   }
   if ($OldBitmap.Width -lt $OldBitmap.Height) { $LongSide=$OldBitmap.Height }
   
       $newH=$SizeLimit
       $newW=[int]($OldBitmap.Width*$SizeLimit/$OldBitmap.Height)
     } else {
       $newW=$SizeLimit
       $newH=[int]($OldBitmap.Height*$newW/$OldBitmap.Width)
     }
     $g=[System.Drawing.Graphics]::FromImage($NewBitmap)
     $g.InterpolationMode = [System.Drawing.Drawing2D.InterpolationMode]::HighQualityBicubic
 
     $NewBitmap.Dispose()
     $OldBitmap.Dispose()
     $n=get-childitem $name
     }
     $name=$_.DirectoryName+"\"+$_.name.substring(0, $_.name.indexof($_.extension))+".jpg"
     $OldBitmap.Save($name, ([system.drawing.imaging.imageformat]::jpeg))
     $OldBitmap.Dispose()
     $n=get-childitem $name
     } else {
     }
   }
 }
`

