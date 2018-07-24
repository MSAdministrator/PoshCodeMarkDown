---
Author: thomas freudenbe
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5905
Published Date: 2015-06-24t13
Archived Date: 2015-06-26t07
---

# expand-zipfile - 

## Description

extracts the contents of a zip file to a folder

## Comments



## Usage



## TODO



## function

`expand-zipfile`

## Code

`#
 #
 function Expand-ZipFile {
     param {
        $zipPath,
        $destination,
        [switch] $quiet
     }
 
     $shellApplication = new-object -com shell.application
     $zipPackage = $shellApplication.NameSpace($zipPath)
     $destinationFolder = $shellApplication.NameSpace($destination) 
 
     #$destinationFolder.CopyHere($zipPackage.Items(),20) 
     $total = $zipPackage.Items().Count
     $progress = 1;
     foreach ($zipFile in $zipPackage.Items()) {
         if(!$quiet) {
             Write-Progress "Extracting $zipPath" "Extracting $($zipFile.Name)" -id 0 -percentComplete (($progress/$total)*100)        
         }
 
         $destinationFolder.CopyHere($zipFile,20) 
         $progress++
     }
     if(!$quiet) {
         Write-Progress "Extracted $zipPath" "Extracted $total items" -id 0
     }
 }
`

