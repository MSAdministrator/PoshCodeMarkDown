---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 1.1
Encoding: ascii
License: cc0
PoshCode ID: 168
Published Date: 2009-04-09t12
Archived Date: 2016-03-15t16
---

# new-script - 

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
    [int[]]$id=@((Get-History -count 1| Select Id).Id)
 )
 
 $commands = &{if($id.Count -gt 1){ Get-History -id $id } else { Get-History -count $count }} | &{process{ $_.CommandLine }}
 
 if($script -eq "clipboard") {
    if( @(Get-PSSnapin -Name "pscx").Count ) {
       $commands | out-clipboard
    }elseif(@(gcm clip.exe).Count) {
       $commands | clip
    }
 } else {
    $folder = Split-Path $script
    if(!$folder) {
       $folder = Join-Path (Split-Path $Profile) "Scripts"
    }
    if(!(Test-Path $folder)) { 
       Throw (new-object System.IO.DirectoryNotFoundException "Cannot find path '$folder' because it does not exist.")
    }
    $file = Join-Path $folder (Split-Path $script -leaf)
    if(!(([IO.FileInfo]$file).Extension)) { 
       $file = "$file.ps1"
    }
    $commands | set-content $file
 }
 #}
`

