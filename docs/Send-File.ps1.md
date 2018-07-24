---
Author: lee holmes
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2216
Published Date: 2010-09-09t21
Archived Date: 2016-10-10t09
---

# send-file.ps1 - 

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
 
 Sends a file to a remote session.
 
 .EXAMPLE
 
 PS >$session = New-PsSession leeholmes1c23
 PS >Send-File c:\temp\test.exe c:\temp\test.exe $session
 
 #>
 
 param(
     [Parameter(Mandatory = $true)]
     $Source,
 
     [Parameter(Mandatory = $true)]
     $Destination,
 
     [Parameter(Mandatory = $true)]
     [System.Management.Automation.Runspaces.PSSession] $Session
 )
 
 Set-StrictMode -Version Latest
 
 $sourcePath = (Resolve-Path $source).Path
 $sourceBytes = [IO.File]::ReadAllBytes($sourcePath)
 $streamChunks = @()
 
 Write-Progress -Activity "Sending $Source" -Status "Preparing file"
 $streamSize = 1MB
 for($position = 0; $position -lt $sourceBytes.Length;
     $position += $streamSize)
 {
     $remaining = $sourceBytes.Length - $position
     $remaining = [Math]::Min($remaining, $streamSize)
 
     $nextChunk = New-Object byte[] $remaining
     [Array]::Copy($sourcebytes, $position, $nextChunk, 0, $remaining)
     $streamChunks += ,$nextChunk
 }
 
 $remoteScript = {
     param($destination, $length)
 
     $Destination = $executionContext.SessionState.`
         Path.GetUnresolvedProviderPathFromPSPath($Destination)
 
     $destBytes = New-Object byte[] $length
     $position = 0
 
     foreach($chunk in $input)
     {
         Write-Progress -Activity "Writing $Destination" `
             -Status "Sending file" `
             -PercentComplete ($position / $length * 100)
 
         [GC]::Collect()
         [Array]::Copy($chunk, 0, $destBytes, $position, $chunk.Length)
         $position += $chunk.Length
     }
 
     [IO.File]::WriteAllBytes($destination, $destBytes)
 
     Get-Item $destination
     [GC]::Collect()
 }
 
 $streamChunks | Invoke-Command -Session $session $remoteScript `
     -ArgumentList $destination,$sourceBytes.Length
`

