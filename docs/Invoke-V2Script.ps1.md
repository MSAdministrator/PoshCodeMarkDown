---
Author: bartekb
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2963
Published Date: 2012-09-23t06
Archived Date: 2012-01-14t07
---

# invoke-v2script - 

## Description

function may be useful for people who want to play with ctp1 for powershell 3 but need to use v2 scripts (and prefer ise over powershell.exe)

## Comments



## Usage



## TODO



## function

`invoke-v2script`

## Code

`#
 #
 
 function Invoke-V2Script {
 
 <#
     .Synopsis
         Will run script currently edited in ISE using powershell V2.
     .Description
         Will run script currently edited ISE. If it's not saved - it will automatically save it.
         Difference between this command and F5 is that script will be launched using powershell V2.
     .Example
         Invoke-Script
         Will run a script/ scriptblock using currently opened file.
 #>
     
     [CmdletBinding()]
     param ()
 
     if ($file = $psISE.CurrentFile) {
         if (Test-Path $file.FullPath) {
             Write-Verbose 'File exists...'
             if (-not $File.IsSaved) {
                 $file.Save()
             }
         } else {
             Write-Verbose 'Save a file as random .ps1 and run'
             $NewName = Join-Path ([io.path]::GetTempPath()) ([io.path]::GetRandomFileName() + '.ps1')
             $file.SaveAs($NewName)
             $FullName = (Get-Item $NewName).FullName
             powershell -Version 2 -NoProfile -File $FullName
             return
         } 
         if ((Get-Item $file.FullPath).extension -eq '.ps1') {
             Write-Verbose 'Normal script, need to run it!'
             powershell -Version 2 -NoProfile -File $file.FullPath
         }
         
     } else {
         throw 'No file opened in current tab or tried to use it outside PowerShell ISE!'
     }
 }
 
 try {
     $psISE.CurrentPowerShellTab.AddOnsMenu.Submenus.Add('Invoke V2 Script', { Invoke-V2Script }, 'CTRL + F5')
 } catch {
     Write-Warning 'Could not add shortcut. Adding menu item without it...'
     $psISE.CurrentPowerShellTab.AddOnsMenu.Submenus.Add('Invoke V2 Script', { Invoke-V2Script }, $null)
 }
`

