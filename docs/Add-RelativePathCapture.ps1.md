---
Author: lee holmes
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2131
Published Date: 2011-09-09t21
Archived Date: 2016-05-17t08
---

# add-relativepathcapture. - 

## Description

from windows powershell cookbook (o’reilly) by lee holmes

## Comments



## Usage



## TODO



## 

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
 
 Adds a new Out-Default command wrapper that captures relative path
 navigation without having to explicitly call 'Set-Location'
 
 .EXAMPLE
 
 PS C:\Users\Lee\Documents>..
 PS C:\Users\Lee>...
 PS C:\>
 
 .NOTES
 
 This commands builds on New-CommandWrapper, also included in the Windows
 PowerShell Cookbook.
 
 #>
 
 Set-StrictMode -Version Latest
 
 New-CommandWrapper Out-Default `
     -Process {
         if(($_ -is [System.Management.Automation.ErrorRecord]) -and
             ($_.FullyQualifiedErrorId -eq "CommandNotFoundException"))
         {
             $command = $_.TargetObject
             if($command -match '^(\.)+$')
             {
                 $newLocation = "..\" * ($command.Length - 1)
                 if($newLocation) { Set-Location $newLocation }
 
                 $error.RemoveAt(0)
                 $_ = $null
             }
         }
     }
`

