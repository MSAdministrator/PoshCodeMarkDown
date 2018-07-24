---
Author: lee holmes
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2219
Published Date: 2011-09-09t21
Archived Date: 2016-07-20t17
---

# set-clipboard.ps1 - 

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
 #############################################################################
 ##
 ##
 ##
 ##############################################################################
 
 <#
 
 .SYNOPSIS
 
 Sends the given input to the Windows clipboard.
 
 .EXAMPLE
 
 dir | Set-Clipboard
 This example sends the view of a directory listing to the clipboard
 
 .EXAMPLE
 
 Set-Clipboard "Hello World"
 This example sets the clipboard to the string, "Hello World".
 
 #>
 
 param(
     [Parameter(ValueFromPipeline = $true)]
     [object[]] $InputObject
 )
 
 begin
 {
     Set-StrictMode -Version Latest
     $objectsToProcess = @()
 }
 
 process
 {
     $objectsToProcess += $inputObject
 }
 
 end
 {
     $objectsToProcess | PowerShell -NoProfile -STA -Command {
         Add-Type -Assembly PresentationCore
 
         $clipText = ($input | Out-String -Stream) -join "`r`n"
 
         [Windows.Clipboard]::SetText($clipText)
     }
 }
`

