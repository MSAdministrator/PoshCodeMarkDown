---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2314
Published Date: 
Archived Date: 2010-10-21t23
---

# set-extension - 

## Description

change the extension on files

## Comments



## Usage



## TODO



## function

`set-extension`

## Code

`#
 #
 function Set-Extension { 
 #.Synopsis
 #.Example
 #
 #.Example
 #
 #.Example
 #
 #.Parameter Path
 #.Parameter Extension
 #.Parameter Rename
 param(
    [Parameter(ValueFromPipeline=$true,ValueFromPipelineByPropertyName=$true,Mandatory=$true)]
    [Alias("FullName")]
    [string]$Path
 ,
    [Parameter(Mandatory=$true,Position=0)]
    [string]$Extension
 ,
    [Parameter()]
    [switch]$Rename
 ,
    [Parameter()]
    [switch]$Passthru
 ) 
 process { 
    if($Rename) {
       Move-Item -Literal $Path -Destination ([IO.Path]::ChangeExtension($path, $extension)) -Passthru:$Passthru
    } else {
       [IO.Path]::ChangeExtension($path, $extension) 
    }
 }
 }
`

