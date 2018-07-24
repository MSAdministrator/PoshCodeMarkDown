---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 425
Published Date: 
Archived Date: 2011-01-20t22
---

# exif query script - 

## Description

‘get-exif’ doesn’t seem to exist. there are various solutions suggested for this, but i could not find a way to get fnumber and focal length in 35mm.  so i have parsed out ‘exif’ output in this script.

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 
 
 $exif_index  =  gci *.jpg | %{exif ($_.Name)}
 $c = $exif_index | Select-String -pattern "EXIF tag" , FNumber , "Focal Length In 35mm"
 $c1 =  ("$c").Split(  ) | Select-String -pattern JPG , f/ , mm
 $c2 = (("$c1").Replace( "'" , "")).Split()
 $c3 = (("$c2").Replace( " |f/" , ",")).Split()
 $c4 = (("$c3").Replace( " 35mm|" , ",")).Split()
 
 if ((gci PhotoData.csv).Exists -eq "True") {mv PhotoData.csv PhotoData.csv.old -force}
 else {}
 
 "Exif_Tag,FNumber,Focal_Length" | out-file PhotoData.csv
 $c4 | out-file -append PhotoData.csv
 $PhotoData = import-csv -path PhotoData.csv
 
 $FileName =  ( $PhotoData | Measure-Object -Property Exif_Tag  )
 $FNumber = ( $PhotoData | Measure-Object -Property FNumber  -average -maximum -minimum ) 
 $Focal_Length = ( $PhotoData | Measure-Object -Property Focal_Length  -average -maximum -minimum ) 
 
 echo $FileName
 echo $FNumber
 echo $Focal_Length
`

