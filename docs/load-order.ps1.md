---
Author: greg zakharov
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6723
Published Date: 2017-02-07t13
Archived Date: 2017-02-11t03
---

# load order - 

## Description

original post found [here](https

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 <#
   .SYNOPSIS
       Analogue of Sysinternals LoadOrd tool.
   .NOTES
       Author: greg zakharov
 #>
 $root = 'HKLM:\SYSTEM\CurrentControlSet\Control'
 $type, $list = (Get-ItemProperty "$($root)\ServiceGroupOrder").List, (
   Get-ItemProperty "$($root -replace 'control\Z', 'services')\*" |
   Where-Object { [Int32]$_.Start -lt 3 }
 )
 
 $s1, $s2, $s3 = @(), @(), @()
 foreach ($t in $type) {
   if ((
     $obj = $list | Where-Object { $_.Group -eq $t }
   ) -eq $null) { continue }
   
   $obj = if ($obj -is [Array]) {
     $rk = Get-Item "$($root)\GroupOrderList"
     
     $val, $arr = $rk.GetValue($t), @()
     if ($val) {
       for ($i = 0; $i -lt $val.Length; $i += 3) {
         $arr += [BitConverter]::ToUInt16($val[$i..($i + 3)], 0)
         $i++
       }
       $arr = $arr[1..($arr.Length - 1)]
       
       foreach ($a in $arr) {
         $obj | Where-Object { [UInt16]$_.Tag -eq $a }
       }
     }
     
     $rk.Dispose()
   }
   else { $obj }
   
   foreach ($o in $obj) {
     switch ($o.Start) {
       0 { $s1 += $o }
       1 { $s2 += $o }
       2 { $s3 += $o }
     }
   }
 }
 
 $s1 + $s2 + $s3 | Select-Object @{N='Start value';E={
   switch ($_.Start) { 0 {'Boot'}; 1 {'System'}; 2 {'Automatic'} }
 }}, @{N='Group name';E={$_.Group}}, Tag, @{
   N='Service/Device';E={$_.PSChildName}
 }, @{N='Display name';E={$_.DisplayName}}, @{
   N='Image path';E={$_.ImagePath}
 } | Out-GridView -Title LoadOrd
`

