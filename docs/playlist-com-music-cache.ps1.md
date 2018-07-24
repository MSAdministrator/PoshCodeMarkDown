---
Author: karl prosser
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 1437
Published Date: 
Archived Date: 2011-01-18t22
---

# playlist.com music cache - 

## Description

a script to download playlist.com music to a local cache directory for when your internet connection is spotty.

## Comments



## Usage



## TODO



## function

``

## Code

`#
 #
 function calculateurl ([string]$source)
 {
 $encoded =  ([byte[]]([regex]::matches($source,'\w{2}') |% {"0x$_"}))
 $a = 0
 $sbox = 0..255  
 $seed = "sdf883jsdf22";
 $mykey = 0..255 | % {  ([byte[]] ($seed.tochararray()))[$_ % $seed.length] }  
 0..255 | % {$a = ($a + $sbox[$_] + $mykey[$_]) % 256;$b = $sbox[$_];$sbox[$_] = $sbox[$a];$sbox[$a] = $b;}
 $a = $b = 0;$output = "";
 $encoded | % {$a = ($a + 1) % 256;$b = ($b + $sbox[$a]) % 256;$e = $sbox[$a];$sbox[$a] = $sbox[$b];$sbox[$b] = $e;
                      $h = ($sbox[$a] + $sbox[$b]) % 256; $d = $sbox[$h]; $_ -bxor $d } | % { $output += [char]$_ }
 $output
 }
 function cacheplaylist([string]$path,[string]$playlistnumber)
 {
 ([xml](new-object system.net.webclient).downloadstring("http://pl.playlist.com/pl.php?playlist=$([int]($playlistnumber/256))")).playlist.tracklist.track | % {
 try { $a= $_.tracktitle;$a;(new-object system.net.webclient).downloadfile((calculateurl $_.location),"$path\$a.mp3");"$a downloaded" } catch {"$a failed" } } 
 }
  cacheplaylist c:\musiccache 2952517899
`

