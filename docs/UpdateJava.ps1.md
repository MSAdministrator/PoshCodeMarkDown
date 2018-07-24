---
Author: donotnottouch
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 4224
Published Date: 2016-06-24t16
Archived Date: 2016-01-21t17
---

# updatejava - 

## Description

a simple script to download and install java 7 u 25. yeah, you could use chocolatey but i don’t want to install it.

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 
 $64bit = $false
 if(Test-Path 'C:\Program Files (x86)' -PathType Container){
     $64bit = $true
 }
 
 try{
     if(Test-Path 'C:\Download\Java' -PathType Container){
         $destinationx86 ='C:\Download\Java\java7u25x86.exe'
         if($64bit){
             $destinationx64 ='C:\Download\Java\java7u25x64.exe'
         }
     }
     else{
         New-Item -Path 'C:\Download' -Name 'Java' -ItemType directory
         $destinationx86 ='C:\Download\Java\java7u25x86.exe'
         if($64bit){
             $destinationx64 ='C:\Download\Java\java7u25x64.exe'
         }
     }
 
     $wc = New-Object System.Net.WebClient
     $wc.DownloadFile($sourcex86, $destinationx86)
     if($64bit){
         $wc.DownloadFile($sourcex64, $destinationx64)
     }
 }
 catch{
     $Error[0]
     return (-1)
 }
 
 
 
 
 CMD /c $destinationx86 /s
 if($64bit){
     CMD /c $destinationx64 /s
 }
`

