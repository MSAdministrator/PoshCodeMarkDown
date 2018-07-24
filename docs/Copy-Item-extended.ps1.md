---
Author: bernd kriszio
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 3475
Published Date: 2013-06-24t10
Archived Date: 2016-04-20t01
---

# copy-item extended - 

## Description

my replacement of copy-item addresses 3 problems. 1) if $dest doesn’t exist, it creates a directory 2) $exclude works on all levels  3) it handles wildcards in $src (in the filepart, this extends the solution to 2) in http

## Comments



## Usage



## TODO



## function

`copy-tocreatefolder`

## Code

`#
 #
 function Copy-ToCreateFolder
 {
     param(
         [string]$src,
         [string]$dest,
         $exclude,
         [switch]$Recurse
     )
     
     
     if (Test-Path($src))
     {
         [void](New-Item $dest -itemtype directory -EA silentlycontinue )
         Get-ChildItem -Path $src -Force -exclude $exclude | % {
             
             if ($_.psIsContainer)
             {
                 {
                     $sub = $_
                     $p = Split-path $sub
                     $currentfolder = Split-Path $sub -leaf
                     [void](New-item $dest\$currentfolder -type directory -ea silentlycontinue)
                     Get-ChildItem $_ -Recurse:$Recurse -name  -exclude $exclude -Force | % {  Copy-item $sub\$_ $dest\$currentfolder\$_ }
                 }
             }
             else
             {
                 
                 #"{0}    {1}" -f (split-path $_.fullname), (split-path $_.fullname -leaf)
                 Copy-Item $_ $dest
             }
         }
     }
 }
`

