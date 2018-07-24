---
Author: lee holmes
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2202
Published Date: 2011-09-09t21
Archived Date: 2016-03-19t02
---

# new-zipfile.ps1 - 

## Description

from windows powershell cookbook (o’reilly) by lee holmes

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 ##############################################################################
 ##
 ##
 ##
 ##############################################################################
 
 <#
 
 .SYNOPSIS
 
 Create a Zip file from any files piped in. Requires that
 you have the SharpZipLib installed, which is available from
 http://www.icsharpcode.net/OpenSource/SharpZipLib/
 
 .EXAMPLE
 
 dir *.ps1 | New-ZipFile scripts.zip d:\bin\ICSharpCode.SharpZipLib.dll
 Copies all PS1 files in the current directory to scripts.zip
 
 .EXAMPLE
 
 "readme.txt" | New-ZipFile docs.zip d:\bin\ICSharpCode.SharpZipLib.dll
 Copies readme.txt to docs.zip
 
 #>
 
 param(
     $ZipName = $(throw "Specify a zip file name"),
 
     $LibPath = $(throw "Specify the path to SharpZipLib.dll")
 )
 
 Set-StrictMode -Version Latest
 
 [void] [Reflection.Assembly]::LoadFile($libPath)
 $namespace = "ICSharpCode.SharpZipLib.Zip.{0}"
 
 $zipName = $executionContext.SessionState.`
     Path.GetUnresolvedProviderPathFromPSPath($zipName)
 $zipFile =
     New-Object ($namespace -f "ZipOutputStream") ([IO.File]::Create($zipName))
 $zipFullName = (Resolve-Path $zipName).Path
 
 [byte[]] $buffer = New-Object byte[] 4096
 
 foreach($file in $input)
 {
     if($file.FullName -eq $zipFullName)
     {
         continue
     }
 
     $replacePath = [Regex]::Escape( (Get-Location).Path + "\" )
     $zipName = ([string] $file) -replace $replacePath,""
 
     $zipEntry = New-Object ($namespace -f "ZipEntry") $zipName
     $zipFile.PutNextEntry($zipEntry)
 
     $fileStream = [IO.File]::OpenRead($file.FullName)
     [ICSharpCode.SharpZipLib.Core.StreamUtils]::Copy(
         $fileStream, $zipFile, $buffer)
     $fileStream.Close()
 }
 
 $zipFile.Close()
`

