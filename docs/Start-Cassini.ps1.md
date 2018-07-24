---
Author: george mauer
Publisher: 
Copyright: 
Email: 
Version: 10.0
Encoding: ascii
License: cc0
PoshCode ID: 3034
Published Date: 2012-11-01t09
Archived Date: 2012-01-14t07
---

# start-cassini - 

## Description

start up the .net 4.0 cassini webserver. useful for those of us who like to avoid opening visual studio

## Comments



## Usage



## TODO



## function

`start-cassini`

## Code

`#
 #
 function Start-Cassini([string]$physical_path=((@(Coalesce-Args (Find-File Global.asax).DirectoryName "."))[0]), [int]$port=12372, [switch]$dontOpenBrowser) {
   $serverLocation = Resolve-Path "C:\Program Files (x86)\Common Files\Microsoft Shared\DevServer\10.0\WebDev.WebServer40.EXE";
   $full_app_path = Resolve-Path $physical_path
   $url = "http://localhost:$port"
   &($serverLocation.Path) /port:$port /path:"""$($full_app_path.Path)"""
   Write-Host "Started $($full_app_path.Path) on $url"
   Write-Host "Remember you can kill processes with kill -name WebDev*"
   if($dontOpenBrowser -eq $false) {
     [System.Diagnostics.Process]::Start($url)
   }
 }
`

