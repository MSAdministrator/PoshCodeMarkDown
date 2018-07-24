---
Author: karl prosser
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 843
Published Date: 2009-02-03t12
Archived Date: 2016-03-06t11
---

# print-file - 

## Description

simple v1 function to print files, either listed, or through the pipeline. no error checking implemented

## Comments



## Usage



## TODO



## function

`print-file`

## Code

`#
 #
 function print-file($file)
 {
  begin  {               
     function internal-printfile($thefile)
     {    
         if ($thefile -is [string]) {$filename = $thefile } 
         else { 
                 if ($thefile.FullName -is [string] ) { $filename = $THEfile.FullName } 
              }   
         $start = new-object System.Diagnostics.ProcessStartInfo $filename
         $start.Verb = "print"
         [System.Diagnostics.Process]::Start($start)                     
     }
     
 if ($file -ne $null) {
                 $filespecified = $true;
                 internal-printfile $file
             }
        }     
 process{if (!$filespecified) { write-Host process ; internal-printfile $_ } }
 
 }
`

