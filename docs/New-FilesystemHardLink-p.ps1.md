---
Author: lee holmes
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2199
Published Date: 2011-09-09t21
Archived Date: 2016-03-19t02
---

# new-filesystemhardlink.p - 

## Description

from windows powershell cookbook (o’reilly) by lee holmes

## Comments



## Usage



## TODO



## function

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
 
 Create a new hard link, which allows you to create a new name by which you
 can access an existing file. Windows only deletes the actual file once
 you delete all hard links that point to it.
 
 .EXAMPLE
 
 PS >"Hello" > test.txt
 PS >dir test* | select name
 
 Name
 ----
 test.txt
 
 PS >.\New-FilesystemHardLink.ps1 test.txt test2.txt
 PS >type test2.txt
 Hello
 PS >dir test* | select name
 
 Name
 ----
 test.txt
 test2.txt
 
 #>
 
 param(
     [string] $Path,
 
     [string] $Destination
 )
 
 Set-StrictMode -Version Latest
 
 $filename = $executionContext.SessionState.`
     Path.GetUnresolvedProviderPathFromPSPath($destination)
 $existingFilename = Resolve-Path $path
 
 $parameterTypes = [string], [string], [IntPtr]
 $parameters = [string] $filename, [string] $existingFilename, [IntPtr]::Zero
 
 $currentDirectory = Split-Path $myInvocation.MyCommand.Path
 $invokeWindowsApiCommand = Join-Path $currentDirectory Invoke-WindowsApi.ps1
 $result = & $invokeWindowsApiCommand "kernel32" `
     ([bool]) "CreateHardLink" $parameterTypes $parameters
 
 if(-not $result)
 {
     $message = "Could not create hard link of $filename to " +
         "existing file $existingFilename"
     Write-Error $message
 }
`

