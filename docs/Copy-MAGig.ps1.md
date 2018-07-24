---
Author: bernd kriszio
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 3707
Published Date: 2012-10-21t04
Archived Date: 2012-10-28t04
---

# copy-magig - 

## Description

a script which wraps some magic modes for merging folder trees around copy-item. there is �no overwrite� or keep newest version, don�t copy empty folders and some options to control the logging of the operation. it is intended run with powershell v2 and v3. i intend to blog about it after intense testing on http

## Comments



## Usage



## TODO



## function

`copy-magig`

## Code

`#
 #
 function Copy-MAGig
 {
     param(
         [string]$src,
         [string]$dest,
         $exclude,
         [switch]$Recurse,            
         
     )
     
     
     $errident = [math]::max($ident - 3, 0)
     $summaryident = [math]::max($ident - 4, 0)
     $src_pattern = "^$($src -replace '\\', '\\' )\\"
     $src_obj = Get-item $src
     try
     {
         if (Get-Item $src -EA stop)
         {
             if ($summary) {
                 "{0,-$summaryident}{1,-$width} {2} => {3}" -f  '', $src , (@{'false'= '   ';'true'= 'rec' }[[string]$Recurse]), $dest
             }
 
             Get-ChildItem -Path $src -Force -Rec:$Recurse -exclude $exclude | % {
                 if ($src_obj.psIscontainer)
                 {
                     #'src is container'
                     $relativePath =  $_.FullName -replace $src_pattern, ''
                 }
                 else
                 {
                     #'src is file'
                     $relativePath = Split-Path $src -leaf
                 }
                 #"relativePath: $relativePath"
 
                 if ($_.psIsContainer)
                 {
                     {
                         if ((Get-ChildItem $_.FullName |% { if (! $_.PSiscontainer) {$_} }) -or $keepEmptyFolders)
                         {
                             [void](New-item $dest\$relativePath -type directory -ea silentlycontinue)
                         }
                     }
                 }
                 else
                 {
                     $FileDestination = "$dest\$relativePath"
                     try
                     {
                         $fileExists = Test-Path $FileDestination
                     }
                     catch
                     {
                         $fileExists = $false
                     }
                     if ($fileExists -and $newest) {
                         $leavecurrent =  $_.LastWriteTime -le (Get-item $dest\$relativePath).LastWriteTime
                     } else {
                         $leavecurrent = $false
                     }
                     #"{0} {1} {2} {3}" -f $fileExists, $NoReplace, ($isnewer -and $newest), $relativePath
                     if ($log)
                     {
                         if ($fileExists) {
                             if (! $noreplace) {
                                 if ($leavecurrent){
                                     "{0,-$errident}-- {1,-$width} => {2} (dest is current or newer)" -f  '', $_.FullName,  $FileDestination
                                 } else {
                                     "{0,-$ident}{1,-$width} => {2}" -f  '', $_.FullName,  $FileDestination
                                 }
                             }
 
                         } else {
                             "{0,-$ident}{1,-$width} -> {2}" -f  '', $_.FullName,  $FileDestination
                         }
                     }
                     if ( ! ($fileExists -and ($NoReplace -or $leavecurrent))  )  {
                         try {
                             Copy-Item $_.FullName $dest\$relativePath  -Force
                         } catch {
                         }
                     }
                 }
 
             }
         }
     }
     catch
     {
     }
 }
`

