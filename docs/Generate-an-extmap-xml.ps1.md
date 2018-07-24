---
Author: chui tey
Publisher: 
Copyright: 
Email: 
Version: 1.0
Encoding: ascii
License: cc0
PoshCode ID: 4214
Published Date: 2013-06-22t09
Archived Date: 2013-06-25t00
---

# generate an extmap.xml - 

## Description

generate an extmap.xml for a dll. this is used for assembly library caching in silverlight.

## Comments



## Usage



## TODO



## module

``

## Code

`#
 #
 function ToExtMap($assemblyPath) {
    $fullPath = (Get-ChildItem -filter $assemblyPath).FullName
    $fileName = (Get-ChildItem -filter $assemblyPath).Name
    $assembly = [System.Reflection.AssemblyName]::GetAssemblyName($fullPath)
    $assemblyName = $assembly.Name
    $assemblyVersion = $assembly.Version
    $publicKeyToken = -join($assembly.GetPublicKeyToken() | % {$_.ToString("X2") })
    @"
 <?xml version="1.0" encoding="utf-8" ?>
 <manifest xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:xsd="http://www.w3.org/2001/XMLSchema">
 
   <assembly>
     <name>$assemblyName</name>
     <version>$assemblyVersion</version>
     <publickeytoken>$publicKeyToken</publickeytoken>
     <relpath>$fileName</relpath>
     <extension downloadUri="$assemblyName.zip" />
   </assembly>
 
 </manifest>
 "@
 
 }
`

