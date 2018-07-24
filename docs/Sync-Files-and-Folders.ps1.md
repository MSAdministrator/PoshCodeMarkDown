---
Author: syncfolder
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6240
Published Date: 2017-02-28t11
Archived Date: 2017-03-16t01
---

# sync files and folders - 

## Description

sync files and folders with powershell

## Comments



## Usage



## TODO



## script

`copy-latestfile`

## Code

`#
 #
 param ( 
    [Parameter(Mandatory=$true,HelpMessage= "Enter souce folder path")] 
    [ValidateScript({Test-Path $_ -PathType Container})] 
    [string]$Src, 
    [Parameter(Mandatory=$true,HelpMessage= "Enter destination folder path")] 
    [ValidateScript({Test-Path $_ -PathType Container})] 
    [string]$Dst 
     
 )
 
 function Copy-LatestFile { 
    Param( 
       [string]$File1, 
       [string]$File2, 
       [switch]$WhatIf 
    ) 
    $File1Date = Get-Item $File1 | Foreach-Object {$_.LastWriteTimeUTC} 
    $File2Date = Get-Item $File2 | Foreach-Object {$_.LastWriteTimeUTC} 
    if ($File1Date -gt $File2Date) { 
       Write-Host "$File1 is newer than $File2, performing copy." 
       if($WhatIf) { 
          Copy-Item $File1 $File2 -Force -WhatIf 
       } else { 
          Copy-Item $File1 $File2 -Force 
       } 
    } else { 
       Write-Host "$File2 is newer than $File1, performing copy." 
       if ($whatif) { 
          Copy-Item $File2 $File1 -Force -WhatIf 
       } else { 
          Copy-Item $File2 $File1 -Force 
       } 
    } 
    Write-Host 
 } 
 if(-Not(Test-Path $Destination)) { 
    New-Item $Destination -Type Directory -Force | Out-Null 
 } 
 
 $SrcEntries = Get-ChildItem $Src -Recurse
 $DstEntries = Get-ChildItem $Dst -Recurse
 
 $SrcFolders = $SrcEntries | Where-Object{$_.PSIsContainer} 
 $SrcFiles = $SrcEntries | Where-Object{!$_.PSIsContainer} 
 $DstFolders = $DstEntries | Where-Object{$_.PSIsContainer} 
 $DstFiles = $DstEntries | Where-Object{!$_.PSIsContainer} 
 
 foreach ($Folder in $Srcfolders) { 
    $SrcFolderPath = $Source -replace "\\","\\" -replace "\:","\:" 
    $DstFolder = $Folder.Fullname -replace $SrcFolderPath,$Destination 
    if($DstFolder -ne "") { 
       if(-Not(Test-Path $DstFolder)) { 
          Write-Warning "Folder $DstFolder does not exist. Creating $DstFolder." 
          New-Item $DstFolder -Type Directory | Out-Null 
       } 
    } 
 } 
 
 foreach ($Folder in $DestFolders) { 
    $DstFilePath = $Destination -replace "\\","\\" -replace "\:","\:" 
    $SrcFolder = $Folder.Fullname -replace $DstFilePath,$Source 
    if ($srcFolder -ne "") { 
       if(-Not(Test-Path $SrcFolder)) { 
          Write-Warning "Folder $SrcFolder does not exist. Creating $SrcFolder." 
          New-Item $SrcFolder -Type Directory | Out-Null 
       } 
    } 
 } 
 
 foreach ($Entry in $SrcFiles) { 
    $SrcFullname = $Entry.FullName 
    $SrcName = $Entry.Name 
    $SrcFilePath = $Source -replace "\\","\\" -replace "\:","\:" 
    $DstFile = $SrcFullname -replace $SrcFilePath,$Destination 
    if(Test-Path $DstFile) { 
       $SrcMD5 = (Get-FileHash $SrcFullname).Hash 
       $DstMD5 = (Get-FileHash $DstFile).Hash 
       if ($SrcMD5 -ne $DstMD5) { 
          Write-Warning "File MD5 checksums do not match, deciding by last modified date." 
          Write-Host "MD5: $SrcMD5 File: $SrcFullname" 
          Write-Host "MD5: $DstMD5 File: $DstFile" 
          Copy-LatestFile $SrcFullname $DstFile 
       } 
    } else { 
       Write-Host "$DstFile does not exist, copying from $SrcFullname" 
       Copy-Item $SrcFullName $DstFile -Force 
    } 
 } 
 
 foreach ($Entry in $DesFiles) { 
    $DstFullname = $entry.FullName 
    $DstName = $Entry.Name 
    $DstFilePath = $Destination -replace "\\","\\" -replace "\:","\:" 
    $SrcFile = $DstFullname -replace $DstFilePath,$Source 
    if ($SrcFile -ne "") { 
       if (-Not(Test-Path $SrcFile)) { 
          Write-Host "File $SrcFile does not exist, copying from $DstFullname." 
          Copy-Item $DstFullname $SrcFile -Force 
       } 
    } 
 }
`

