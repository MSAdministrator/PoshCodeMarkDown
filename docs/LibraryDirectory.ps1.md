---
Author: lee holmes
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2190
Published Date: 2011-09-09t21
Archived Date: 2016-03-18t21
---

# librarydirectory.ps1 - 

## Description

from windows powershell cookbook (o’reilly) by lee holmes

## Comments



## Usage



## TODO



## function

`get-directorysize`

## Code

`#
 #
 
 Set-StrictMode -Version Latest
 
 function Get-DirectorySize
 {
     <#
 
     .EXAMPLE
 
     PS > $DebugPreference = "Continue"
     PS > Get-DirectorySize
     DEBUG: Current Directory: D:\lee\OReilly\Scripts\Programs
     Directory size: 46,581 bytes
     PS > $DebugPreference = "SilentlyContinue"
     PS > $VerbosePreference = "Continue"
     PS > Get-DirectorySize
     VERBOSE: Getting size
     VERBOSE: Got size: 46581
     Directory size: 46,581 bytes
     PS > $VerbosePreference = "SilentlyContinue"
 
     #>
 
     Write-Debug "Current Directory: $(Get-Location)"
 
     Write-Verbose "Getting size"
     $size = (Get-ChildItem | Measure-Object -Sum Length).Sum
     Write-Verbose "Got size: $size"
 
     Write-Host ("Directory size: {0:N0} bytes" -f $size)
 }
 
 function Get-ChildItemSortedByLength($path = (Get-Location), [switch] $Problematic)
 {
     <#
     
     .EXAMPLE
 
     PS > Get-ChildItemSortedByLength -Problematic
     out-lineoutput : Object of type "Microsoft.PowerShell.Commands.Internal.Fo
     rmat.FormatEntryData" is not legal or not in the correct sequence. This is
     likely caused by a user-specified "format-*" command which is conflicting
     with the default formatting.
 
     #>
 
     if($Problematic)
     {
         Get-ChildItem $path | Format-Table | Sort Length
     }
     else
     {
         Get-ChildItem $path | Sort Length | Format-Table
     }
 }
`

