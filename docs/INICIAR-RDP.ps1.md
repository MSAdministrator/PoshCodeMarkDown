---
Author: administrator
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 3439
Published Date: 2012-05-30t07
Archived Date: 2012-06-06t07
---

# iniciar-rdp - 

## Description

function/script to launch remote desktop sessions from command line, rdp file or pipeline using terminal services client.

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 ########################################################################################################################
 #
 #
 #
 #
 #
 #
 #
 #
 #
 #
 #
 #
 #
 #
 #
 #
 #
 #
 #
 #
 #
 #
 #
 #
 #
 #
 #
 #
 #
 #
 #
 #
 #
 #
 #
 #
 #
 #
 #
 #
 #
 #
 #
 #
 #
 
     param(
         [string]$Server = "",
         [int]$Width = "",
         [int]$Height = "",
         [string]$Path = "",
         [switch]$Console,
         [switch]$Admin,
         [switch]$Fullscreen,
         [switch]$Public,
         [switch]$Span
     )
 
     begin {
         $arguments = ""
         $dimensions = ""
         $processed = $false
 
         if ($admin) {
             $arguments += "/admin "
         } elseif ($console) {
             $arguments += "/console "
         }
         if ($fullscreen) {
             $arguments += "/f "
         }
         if ($public) {
             $arguments += "/public "
         }
         if ($span) {
             $arguments += "/span "
         }
 
         if ($width -and $height) {
             $dimensions = "/w:$width /h:$height"
         }
     }
 
     process {
         Function script:executePath([string]$path) {
             Invoke-Expression "mstsc.exe '$path' $dimensions $arguments"
         }
         Function script:executeArguments([string]$Server) {
             Invoke-Expression "mstsc.exe /v:$server $dimensions $arguments"
         }
 
         if ($_) {
             if ($_ -is [string]) {
                 if ($_ -imatch '\.rdp$') {
                     if (Test-Path $_) {
                         executePath $_
                         $processed = $true
                     } else {
                         throw "Path does not exist."
                     }
                 } else {
                     executeArguments $_
                     $processed = $true
                 }
             } elseif ($_ -is [System.IO.FileInfo]) {
                 if (Test-Path $_.FullName) {
                     executePath $_.FullName
                     $processed = $true
                 } else {
                     throw "Path does not exist."
                 }
             } elseif ($_.Path) {
                 if (Test-Path $_.Path) {
                     executePath $_.Path
                     $processed = $true
                 } else {
                     throw "Path does not exist."
                 }
             } elseif ($_.DnsName) {
                 executeArguments $_.DnsName
                 $processed = $true
             } elseif ($_.Server) {
                 executeArguments $_.Server
                 $processed = $true
             } elseif ($_.ServerName) {
                 executeArguments $_.ServerName
                 $processed = $true
             } elseif ($_.Name) {
                 executeArguments $_.Name
                 $processed = $true
             }
         }
     }
 
     end {
         if ($path) {
             if (Test-Path $path) {
                 Invoke-Expression "mstsc.exe '$path' $dimensions $arguments"
 
             } else {
                 throw "Path does not exist."
             }
         } elseif ($server) {
             Invoke-Expression "mstsc.exe /v:$server $dimensions $arguments"
         } elseif (-not $processed) {
             Invoke-Expression "mstsc.exe $dimensions $arguments"
         }
     }
 #}
`

