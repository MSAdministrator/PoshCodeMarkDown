---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 686
Published Date: 
Archived Date: 2009-01-05t17
---

# convertto-googlechartnum - 

## Description

a rounding encoder for google charts data

## Comments



## Usage



## TODO



## function

`convertto-googlechartnum`

## Code

`#
 #
 #########################################################################
 BEGIN {
    $ody = "A","B","C","D","E","F","G","H","I","J","K","L","M","N","O","P","Q","R","S","T","U","V","W","X","Y","Z","a","b","c","d","e","f","g","h","i","j","k","l","m","n","o","p","q","r","s","t","u","v","w","x","y","z","0","1","2","3","4","5","6","7","8","9","-","."
 
    filter encode {
       if($_ -ge ($ody.Count * $ody.Count) ) { return "__" } 
       $x = [Math]::DivRem( $_, $ody.Count, [ref]$y )
       return "$($ody[$x])$($ody[$y])"
    }
    [int[]]$nums = $args | % { [int]$_ }
 }
 PROCESS {
    if($_ -ne $null) { $nums += $_ }
 }
 #}
 
 END {
    $diff =  ($nums | sort  | select -last 1) / ($ody.Count * $ody.Count -1)
    $nums | %{$_/$diff} | encode
 }
`

