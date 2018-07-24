---
Author: greg zakharov
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 4134
Published Date: 2013-04-26t12
Archived Date: 2016-06-08t09
---

# directx mp3player - 

## Description

i found managed directx on one of my computers, so i thought write about how to play mp3 files with it. but note, this script has been tested on winxp only. for example

## Comments



## Usage



## TODO



## function

`invoke-mp3player`

## Code

`#
 #
   param (
     [Parameter(Mandatory=$true, Position=0)]
     [string]$MP3File,
 
     [Parameter(Mandatory=$false, Position=1)]
     [int]$Volume = -1000
   )
 
   $dir = (gci (Split-Path (
                 Split-Path ([Runtime.InteropServices.RuntimeEnvironment]::GetRuntimeDirectory())
               )
          ) | ? {$_.Name -cmatch 'DirectX'}).FullName
   if ($dir -eq $null) { break }
   $dir = (gci $dir -r -i 'Microsoft.DirectX.AudioVideoPlayback.dll').FullName
   [void][Reflection.Assembly]::LoadFile($dir)
 
   try {
     $mp3 = [Microsoft.DirectX.AudioVideoPlayback.Audio]::FromFile($MP3File)
     $mp3.Volume = $Volume
 
     $ct = $host.UI.RawUI.WindowTitle
     $ts = New-Object TimeSpan 0, 0, 0
 
     do {
       Start-Sleep -s 1
       $ts += New-Object TimeSpan 0, 0, 1
       [Console]::Title = $ts
       $mp3.Play()
     } while ($mp3.CurrentPosition -ne $mp3.Duration)
     $mp3.Stop()
     $mp3.Dispose()
     $host.UI.RawUI.WindowTitle = $ct
   }
   catch { Write-Host An error has been occurred. Probably`, file is missed.`n }
 #}
`

