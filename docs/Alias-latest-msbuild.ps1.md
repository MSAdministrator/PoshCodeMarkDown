---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 896
Published Date: 
Archived Date: 2009-03-01t00
---

# alias latest msbuild - 

## Description

this is just a little trick i use to find installutil and msbuild, and make sure i’m using the latest version of them.

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 $rtr = Split-Path $([System.Runtime.InteropServices.RuntimeEnvironment]::GetRuntimeDirectory())
 
 foreach($rtd in get-childitem $rtr -filt v* | sort Name) {
    if( Test-Path (join-path $rtd.FullName installutil.exe) ) {
       set-alias installutil (resolve-path (join-path $rtd.FullName installutil.exe))
    }
    if( Test-Path (join-path $rtd.FullName msbuild.exe) ) {
       set-alias msbuild (resolve-path (join-path $rtd.FullName msbuild.exe))
    }
 }
`

