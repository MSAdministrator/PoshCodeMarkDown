---
Author: roflrolle
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6537
Published Date: 2016-09-29t09
Archived Date: 2016-11-08t06
---

# delete ransomware files - 

## Description

script to delete ransomware files

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 $scriptPath = split-path -parent $MyInvocation.MyCommand.Definition
 
 Import-Module "$($scriptPath)\AlphaFS.dll"
 
 $list=@(
 "\\server1\folder1",
 "\\server2\folder2\folder22"
 )
 
 $only_output=$FALSE
 
 if($only_output){
     Write-Host ""
     Write-Host "Only Output! Nothing is deleted really!" -ForegroundColor Magenta
     Write-Host ""
 }
 
 $extension=".odin"
 
 $html_file="HOWDO_text.html"
 
 $logfile="$($scriptPath)\crypto.log"
 
 "$(Get-date)"|Out-File $logfile
 
 $count=0
 
 foreach($folder in $list){
 
     Write-Host "--- $($folder)" -ForegroundColor Yellow
 
     [Alphaleonis.Win32.Filesystem.Directory]::EnumerateFileSystemEntries($folder, '*', [System.IO.SearchOption]::AllDirectories)|Where-Object{$_ -like "*$($extension)" -or $_ -like "*$($html_file)"}|ForEach-Object{
         
         Write-Host "$($_)" -ForegroundColor Yellow
         
         $count+=1
 
         "$($_)"|Out-File $logfile -Append
 
         if(!($only_output)){
             [Alphaleonis.Win32.Filesystem.File]::Delete("$($_)", [Alphaleonis.Win32.Filesystem.PathFormat]::FullPath)
         }
     }
 }
 
 Write-Host ""
 Write-Host ""
 Write-Host "$($count) Files found!" -ForegroundColor Yellow
 Write-Host "The count is html and crypt files combined" -ForegroundColor Yellow
 Write-Host ""
`

