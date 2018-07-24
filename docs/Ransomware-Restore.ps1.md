---
Author: roflrolle
Publisher: 
Copyright: 
Email: 
Version: 2016.09.28
Encoding: ascii
License: cc0
PoshCode ID: 6536
Published Date: 2017-09-29t09
Archived Date: 2017-04-30t09
---

# ransomware restore - 

## Description

script to restore files beeing lost during ransomware attack

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
 "\\myserver1\@GMT-2016.09.28-09.10.00\Folder1",
 "\\myserver2\de\WinApps\@GMT-2016.09.28-12.10.00\Folder2"
 )
 
 $only_output=$FALSE
 
 if($only_output){
     Write-Host ""
     Write-Host "Only Output! Nothing is copied really!" -ForegroundColor Magenta
     Write-Host ""
 }
 
 foreach($ShadowPath in $list){
 
     $restorepath=""
     $split=$($ShadowPath -split "\@")
     $restorepath="$($split[0])$($split[1].Substring($($split[1].IndexOf("\"))+1))"
 
     $count=0
 
     Write-Host ""
     Write-Host "`tRestoring $($restorepath)" -ForegroundColor Yellow
     Write-Host ""
 
     Read-Host -Prompt "Continue ? (press any key)"
 
     $ShadowFiles = @([Alphaleonis.Win32.Filesystem.Directory]::GetFiles($($ShadowPath), '*', [System.IO.SearchOption]::AllDirectories))
 
     foreach($file in $ShadowFiles){
     
         $original_file=""
         $original_file=$($file.Replace("$($ShadowPath)","$($restorepath)"))
         
         $original_folder=$original_file.Replace("$($($original_file.Split("\")[-1]))","")
 
         if(!([Alphaleonis.Win32.Filesystem.Directory]::Exists("$($original_folder)"))){
             if(!($only_output)){
                 [Alphaleonis.Win32.Filesystem.Directory]::CreateDirectory("$($original_folder)")|Out-Null
             }
         }
 
         if(!([Alphaleonis.Win32.Filesystem.File]::Exists("$($original_file)"))){
             
             Try{
                 if(!($only_output)){
                     [Alphaleonis.Win32.Filesystem.File]::Copy($File, $original_file, $True)
                 }
                 
                 Write-Host "$($original_file.Split("\")[-1])" -ForegroundColor Green
                 $count+=1
             }
             Catch{
                 Write-Host "$($original_file.Split("\")[-1])" -ForegroundColor Red
             }
         }
     }
 
     Write-Host ""
     Write-Host "$($count) Files restored"
     Write-Host ""
 
 }
`

