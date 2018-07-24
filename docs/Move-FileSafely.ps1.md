---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 1943
Published Date: 2010-06-29t21
Archived Date: 2016-10-18t09
---

# move-filesafely - 

## Description

a wrapper around move-item which moves a whole folder tree and creates backup copies if the files already exist in the destination.

## Comments



## Usage



## TODO



## function

`test-createablepath`

## Code

`#
 #
 function global:Move-FileSafely {
 #.Synopsis
 #.Description 
 #.Parameter Source
 #.Parameter Destination
 #.Parameter BackupCopies
 param($Source, $Destination, $BackupCopies=$(Join-Path $destination (Get-Date -f "backup yyyy-MM-dd")))
    function Test-CreateablePath {
       param($path)
       try {
          return Join-Path (Resolve-Path (Split-Path $path) -ErrorAction STOP) (Split-Path $path -Leaf)
       } catch {
          return (split-path $path -isabsolute)
       }
    }
    
 
    if(!(Test-CreateablePath $Destination)) {
       throw "Destination ($Destination) path must be a full path, or a subfolder of an existing folder"
    }
    if(!(Test-CreateablePath $BackupCopies)) {
       throw "BackupCopies ($BackupCopies) path must be a full path, or a subfolder of an existing folder"
    }
 
    mkdir $destination -Force -ErrorAction SilentlyContinue | Out-Null
    
    Get-ChildItem $source -recurse -ErrorAction SilentlyContinue | Move-Item -Destination { split-path $_.FullName.Replace($Source, $Destination) } -ErrorVariable Problems -ErrorAction SilentlyContinue
    foreach($file in $Problems | Select -Expand TargetObject | Where { Test-Path $_.FullName -Type Leaf }) { 
       $Target = $File.FullName.Replace($source, $backupCopies)
       mkdir (Split-Path $Target) -Force -ErrorAction SilentlyContinue | Out-Null
       move-item $File.FullName $Target
    }
    remove-item $source -Recurse -Force
 }
`

