---
Author: redyey
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5703
Published Date: 2015-01-22t02
Archived Date: 2015-01-24t06
---

# release-comobject - 

## Description

author unknown.

## Comments



## Usage



## TODO



## function

`remove-comobject`

## Code

`#
 #
 function Remove-ComObject {
  [CmdletBinding()]
  param()
  end {
   Start-Sleep -Milliseconds 500
   [Management.Automation.ScopedItemOptions]$scopedOpt = 'ReadOnly, Constant'
   Get-Variable -Scope 1 | Where-Object {
    $_.Value.pstypenames -contains 'System.__ComObject' -and -not ($scopedOpt -band $_.Options)
   } | Remove-Variable -Scope 1 -Verbose:([Bool]$PSBoundParameters['Verbose'].IsPresent)
   [gc]::Collect()
  }
 }
`

