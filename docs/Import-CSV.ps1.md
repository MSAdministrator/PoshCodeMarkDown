---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 93
Published Date: 
Archived Date: 2010-10-07t08
---

# import-csv.ps1 - 

## Description

import-csv that takes headers

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 Param($file,$headers)
 if($input){$data = @();$input | foreach-Object{$data += $_}}
 
 if($file)
 {
     if(Test-Path $file)
     {
         if(!$headers)
         {
             import-Csv $data
             return
         }
         else
         {
             $data = Get-Content $file
         }
     }
     else{Write-Host "Invalid File Passed";return $false}
 }
        
 foreach($entry in $data)
 {
     $values = $entry.Split(",")
     if($values.Count -gt $headers.Count)
     {
         Write-Host "Value Count [$($values.Count)] and Headers [$($Headers.Count)] do not match"
         return $false
     }
     $myobj = "" | Select-Object -prop $headers
     $i = 0
     foreach($value in $values)
     {
         $header = $headers[$i]
         $myobj.$header = $value
         $i++
     }
     $myobj
 }
`

