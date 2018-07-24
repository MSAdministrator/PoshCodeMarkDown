---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 1.1
Encoding: ascii
License: cc0
PoshCode ID: 171
Published Date: 2009-04-11t19
Archived Date: 2016-03-04t23
---

# new-script 2 - 

## Description

create a new script from a series of commands in your history… or copy them to the clipboard.

## Comments



## Usage



## TODO



## script

`new-script`

## Code

`#
 #
 ##################################################################################################
 ##################################################################################################
 ##
 ##################################################################################################
 ##################################################################################################
 
 param( 
    [string]$script=(Read-Host "A name for your script"),
    [int]$count=1, 
    [int[]]$id=@((Get-History -count 1| Select Id).Id),
    [switch]$Force
 )
 
 if($id.Count -eq 1) { 1..($count-1)|%{ $id += $id[-1]-1 } }
 $commands = Get-History -id $id | &{process{ $_.CommandLine }}
 
 if($script -eq "clipboard") {
    if( @(Get-PSSnapin -Name "pscx").Count ) {
       $commands | out-clipboard
    } elseif(@(gcm clip.exe).Count) {
       $commands | clip
    }
 } else {
    $folder = Split-Path $script
    if(!$folder) {
       $folder = Join-Path (Split-Path $Profile) "Scripts"
    }
    if(!(Test-Path $folder)) { 
       $folder = Get-Location -PSProvider "FileSystem"
    }
    $file = Join-Path $folder (Split-Path $script -leaf)
    if(!(([IO.FileInfo]$file).Extension)) { 
       $file = "$file.ps1"
    }
    if((Test-Path $file) -and (!$Force)) {
       Write-Error "The file already exists, do you want to overwrite?"
    }
    $commands | set-content $file -Confirm:((Test-Path $file) -and (!$Force))
    Get-Item $file
 }
 #}
`

