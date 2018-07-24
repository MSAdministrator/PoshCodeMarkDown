---
Author: clint jones
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 3450
Published Date: 2012-06-08t11
Archived Date: 2012-06-13t22
---

# report-decomvms - 

## Description

use this to view vm that have been decommissioned and turned off for more than 2 weeks so that they can be deleted.

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 #========================================================================
 #========================================================================
 
 Add-PSSnapin VMware.VimAutomation.Core
 
 Connect-VIServer -Server <viserver> -Credential (Get-Credential)
 
 $deletenow = @()
 $deletesoon = @()
 
 $dnrvms = Get-VM | Where-Object {($_.Name.Contains("DNR")) -and ($_.PowerState -eq "PoweredOff")}
 
 foreach ($dnrvm in $dnrvms)
 {
 
     [array]$poweroffs = Get-VM -Name $dnrvm.Name | Get-VIEvent -Start (Get-Date).AddDays(-14) | Where-Object {$_.FullFormattedMessage -like "*is powered off"}
     
     if ($poweroffs -eq $null)
     {
       $deletenow += $dnrvm.Name
     }
     else
     {
       $deletesoon += $dnrvm.Name
     }
  
 }
 
 $deletesoon = $deletesoon | Select-Object -Unique
 $deletenow = $deletenow | Select-Object -Unique
`

