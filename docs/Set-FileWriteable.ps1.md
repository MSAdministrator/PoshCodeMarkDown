---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2274
Published Date: 
Archived Date: 2010-10-04t16
---

# set-filewriteable - 

## Description

a demo script which work against remote session output as well as local output

## Comments



## Usage



## TODO



## function

`set-filewriteable`

## Code

`#
 #
 function Set-FileWriteable {
 #.Example
 #
 param(
    [Parameter(Mandatory=$true,ValueFromPipeline=$true)]   
    $File
 ,
    [switch]$Passthru
 )
 process {
    foreach($path in @($file)) {
       write-verbose "'$path' is on '$($path.PSComputerName)'"
       if($path.PSComputerName) {
          Invoke-Command $path.PSComputerName {
             param([string[]]$path,[switch]$passthru)
             $files = Get-Item $path
             foreach($f in $files) {
                if($f.Attributes -band [IO.FileAttributes]"ReadOnly") {
                   $f.Attributes = $f.Attributes -bxor [IO.FileAttributes]"ReadOnly"
                }
             }
             write-output $files
          } -Argument $path | Where { $Passthru }
       } else {
          $files = Get-Item $path
          foreach($f in $files) {
             if($f.Attributes -band [IO.FileAttributes]"ReadOnly") {
                $f.Attributes = $f.Attributes -bxor [IO.FileAttributes]"ReadOnly"
             }
          }
          if($Passthru) { write-output $files }
       }
    }
 }
 }
`

