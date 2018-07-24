---
Author: lee holmes
Publisher: 
Copyright: 
Email: 
Version: 1.0
Encoding: ascii
License: cc0
PoshCode ID: 2130
Published Date: 2011-09-09t21
Archived Date: 2016-05-17t13
---

# add-objectcollector.ps1 - 

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
 
 Adds a new Out-Default command wrapper to store up to 500 elements from
 the previous command. This wrapper stores output in the $ll variable.
 
 .EXAMPLE
 
 PS >Get-Command $pshome\powershell.exe
 
 CommandType     Name                          Definition
 -----------     ----                          ----------
 Application     powershell.exe                C:\Windows\System32\Windo...
 
 PS >$ll.Definition
 C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe
 
 .NOTES
 
 This command builds on New-CommandWrapper, also included in the Windows
 PowerShell Cookbook.
 
 #>
 
 Set-StrictMode -Version Latest
 
 New-CommandWrapper Out-Default `
     -Begin {
         $cachedOutput = New-Object System.Collections.ArrayList
     } `
     -Process {
         if($_ -ne $null) { $null = $cachedOutput.Add($_) }
         while($cachedOutput.Count -gt 500) { $cachedOutput.RemoveAt(0) }
     } `
     -End {
         $uniqueOutput = $cachedOutput | Foreach-Object {
             $_.GetType().FullName } | Select -Unique
         $containsInterestingTypes = ($uniqueOutput -notcontains `
             "System.Management.Automation.ErrorRecord") -and
             ($uniqueOutput -notlike `
                 "Microsoft.PowerShell.Commands.Internal.Format.*")
 
         if(($cachedOutput.Count -gt 0) -and $containsInterestingTypes)
         {
             $GLOBAL:ll = $cachedOutput | % { $_ }
         }
     }
`

