---
Author: nathan estell
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6380
Published Date: 2016-06-14t18
Archived Date: 2016-10-06t10
---

# infinite staircase - 

## Description

creates a stepped diagonal line going down to the right, then down to the left.

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 $spaces=0
 for ([int]$rownum=1;;$rownum++) 
     {
     
     $StairDirection=[math]::floor($rownum/50)%2
     if ($StairDirection -eq 0)
         {
         for ($i=0; $i -lt $spaces; $i++) 
             {
             Write-Host " " -NoNewLine
             }
             $spaces++
         }
     if ($StairDirection -eq 1)
         {
             for ($j=0; $j -lt $spaces; $j++) 
                 {
                 Write-Host " " -NoNewLine
                 }
            $spaces--
           if ($spaces -eq -1)
           {
           $spaces=1
           }
         }
         
     
     Write-Host " " -BackgroundColor "Green" 
 }
`

