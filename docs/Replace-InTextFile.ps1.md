---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 1707
Published Date: 2011-03-16t13
Archived Date: 2016-03-05t20
---

# replace-intextfile - 

## Description

a script to do replace strings in text files. yes, this is just a wrapper around (gc) -replace | sc

## Comments



## Usage



## TODO



## function

`creplace-string`

## Code

`#
 #
 param ( 
         [string] $Path = "."
       , [string] $pattern = $(Read-Host "Please enter a pattern to match")
       , [string] $replace = $(Read-Host "Please enter a replacement string")
       , [switch] $recurse = $false
       , [switch] $caseSensitive = $false
 )
 
 if ($pattern -eq $null -or $pattern -eq "") { Write-Error "Please provide a search pattern!" ; return }
 if ($replace -eq $null -or $replace -eq "") { Write-Error "Please provide a replacement string!" ; return }
 
 function CReplace-String( [string]$pattern, [string]$replacement )
 {
    process { $_ -creplace $pattern, $replacement }
 }
 function IReplace-String( [string]$pattern, [string]$replacement )
 {
    process { $_ -ireplace $pattern, $replacement }
 }
 
 $files = Get-ChildItem -Path $Path -recurse:$recurse -filter:$Filter
 
 foreach($file in $files) {
    Write-Host "Processing $($file.FullName)"
    if( $caseSensitive ) {
       $str = Get-Content $file.FullName | CReplace-String $pattern $replace
       Write-Host $str
       Set-Content $file.FullName $str
    } else {
       $str = Get-Content $file.FullName | IReplace-String $pattern $replace
       Write-Host $str
       Set-Content $file.FullName $str
    }
 }
 
 if($files.Length -le 0) { 
    Write-Warning "No matching files found"
 }
`

