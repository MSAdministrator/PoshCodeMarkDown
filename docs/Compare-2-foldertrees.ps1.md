---
Author: bernd kriszio
Publisher: 
Copyright: 
Email: 
Version: 9.0
Encoding: ascii
License: cc0
PoshCode ID: 1827
Published Date: 
Archived Date: 2010-05-11t02
---

# compare 2 foldertrees - 

## Description

a small script to find changed files between 2 to foldertrees. added call to diffmerge (from vs 2008)

## Comments



## Usage



## TODO



## function

`compare-foldertrees`

## Code

`#
 #
 function Compare-Foldertrees 
 {
     param(
         $path1, 
         $path2
         )
 
 
     $len1 = $path1.length 
     $len2 = $path2.length 
 
     . Require-function Get-MD5
 
     Write-Host "====== First path only =======`n"
     gci $path1 -rec | ? {! $_.PSISContainer} | % { 
         $fileName1 = $_.fullName
         $fileName = $fileName1.substring($len1)
         $filename2 = $path2 + $fileName
         #$filename1
         #$filename2
         if (! (Test-Path $filename2))
         {
             "$filename"
         } 
     }
 
     Write-Host "`n====== Second path only =======`n"
 
     gci $path2 -rec | ? {! $_.PSISContainer} | % { 
         $fileName2 = $_.fullName
         $fileName = $fileName2.substring($len2)
         $filename1 = $path2 + $fileName
         #$filename1
         #$filename2
         if (! (Test-Path $filename1))
         {
             "$filename"
         } 
     }
 
 
     Write-Host "`n====== Different =======`n"
 
     gci $path1 -rec | ? {! $_.PSISContainer} | % { 
         $fileName1 = $_.fullName
         $fileName = $fileName1.substring($len1)
         $filename2 = $path2 + $fileName
         #$filename1
         #$filename2
         if ( (Test-Path $filename2))
         {
             $md1 = Get-MD5($filename1)
             $md2 = Get-MD5($filename2)
             if ($md1 -ne $md2)
             {
                 "$filename"
                 & 'C:\Program Files\Microsoft Visual Studio 9.0\Common7\IDE\diffmerge.exe' $fileName1 $filename2 
             }
         } 
     }
 }
`

