---
Author: greg zakharov
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 4864
Published Date: 2014-01-31t06
Archived Date: 2014-02-09t11
---

# get-productkey - 

## Description

from greg�s repository on github.

## Comments



## Usage



## TODO



## function

`get-productkey`

## Code

`#
 #
 function Get-ProductKey([String]$Computer = '.') {
   <#
     .NOTES
         Author: greg zakharov
   #>
   
   begin {
     $reg = 'SOFTWARE\Microsoft\Windows NT\CurrentVersion'
     
     function get([Object[]]$obj) {
       begin {
         $map = "BCDFGHJKMPQRTVWXY2346789"
         $key = ""
       }
       process {
         for ($i = 24; $i -ge 0; $i--) {
           $k = 0
           for ($j = 14; $j -ge 0; $j--) {
             $k = ($k * 256) -bxor $obj[$j]
             $obj[$j] = [Math]::Floor([Double]($k / 24))
             $k = $k % 24
           }
           $key = $map[$k] + $key
 
           if (($i % 5) -eq 0 -and $i -ne 0) {$key = "-" + $key}
         }
       }
       end {$key}
     }
   }
   process {
     try {
       $val = [Byte[]]([wmiclass]('\\' + $Computer + '\root\default:StdRegProv')
           ).GetBinaryValue(2147483650, $reg, 'DigitalProductId').uValue[52..66]
       get $val
     }
     catch [Management.Automation.RuntimeException] {
       if ((Read-Host "Access denied. Get local product key?") -eq 'y') {
         get (gp ('HKLM:\' + $reg)).DigitalProductId[52..66]
       }
     }
   }
   end {}
 }
`

