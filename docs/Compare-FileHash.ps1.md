---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 4787
Published Date: 2014-01-09t21
Archived Date: 2014-01-13t10
---

# compare-filehash - 

## Description

a couple of helper functions to make comparing things easier…

## Comments



## Usage



## TODO



## function

`get-relativefilehash`

## Code

`#
 #
 function Get-RelativeFileHash {
    param(
       [Parameter(Mandatory=$True)]
       $Root,
 
       [ValidateSet("SHA1","SHA256","SHA384","SHA512","MACTripleDES","MD5","RIPEMD160")]
       $Algorithm = "SHA256"
    )
    $Root = Convert-Path $Root
    foreach($hash in Get-ChildItem $Root -Recurse | Get-FileHash -Algorithm:$Algorithm) {
       Add-Member -Input $hash NoteProperty Root $Root
       $hash.Path = $hash.Path.Replace($Root,'').TrimStart("\/")
       $hash
    }
 }
 
 function Compare-FileHash {
    param(
       [Parameter(Mandatory=$True)]
       $LeftPath, 
 
       [Parameter(Mandatory=$True)]
       $RightPath,
 
       [ValidateSet("SHA1","SHA256","SHA384","SHA512","MACTripleDES","MD5","RIPEMD160")]
       $Algorithm = "SHA256"
    )
 
    Compare-Object -Property Hash -Passthru (
       Get-RelativeFileHash $LeftPath -Algorithm:$Algorithm) (
       Get-RelativeFileHash $RightPath -Algorithm:$Algorithm) | 
    Group-Object Path |
    Select-Object `
       @{n="Path"; e={$_.Name}}, 
       @{n="Side"; e={ if($_.Count -eq 1){$_.Group[0].SideIndicator}else{"<>"}} },
       @{n="Root"; e={ if($_.Count -eq 1){
                          @{$_.Group[0].SideIndicator=$_.Group[0].Root}
                       } else {
                          @{
                             $_.Group[0].SideIndicator=$_.Group[0].Root
                             $_.Group[1].SideIndicator=$_.Group[1].Root
                          }
                       }
                     }}
 }
`

