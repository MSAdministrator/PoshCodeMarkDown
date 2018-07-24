---
Author: greg zakharov
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5122
Published Date: 2014-04-28t19
Archived Date: 2014-08-29t23
---

# whatis - 

## Description

looks for online information about file with it extension.

## Comments



## Usage



## TODO



## function

`get-objecttype`

## Code

`#
 #
 if (!(Test-Path alias:whatis)) { Set-Alias whatis Get-ObjectType }
 
 function Get-ObjectType {
   <#
     .NOTES
         Author: greg zakharov
   #>
   param(
     [Parameter(Mandatory=$true, ValueFromPipeline=$true)]
     [ValidateScript({Test-Path $_})]
     [String]$Object
   )
   
   $raw = gi (cvpa $Object)
   if ($raw.PSIsContainer) {
     "{0} - folder path`n" -f $raw
     return
   }
   
   if ([String]::IsNullOrEmpty($raw.Extension)) {
     "File has not an extension.`n"
     return
   }
   
   $raw = 'http://pc.net/extensions/file/' + $raw.Extension.Substring(1, $raw.Extension.Length - 1)
   &(cmd /c ftype (cmd /c assoc .htm).Split('=')[1]).Split('"')[1] $raw
 }
`

