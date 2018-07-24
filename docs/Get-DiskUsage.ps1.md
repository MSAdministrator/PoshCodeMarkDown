---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 721
Published Date: 2009-12-08t17
Archived Date: 2017-01-04t03
---

# get-diskusage - 

## Description

the linux/unix ‘du -sh’ command, ala powershell (faster than other versions of this script, but still ever so much slower than compiled code)

## Comments



## Usage



## TODO



## function

``

## Code

`#
 #
 ###############################
 ###############################
 ##
 function du {
 PARAM($path = "$pwd", [switch]$force, $unit="MB", $round=2, [switch]$total) 
    &{ 
    foreach($dir in (get-childitem $path -force | 
          Where { $_.PSIsContainer -and ($Force -or ($_.Attributes -notmatch "ReparsePoint" -or
                                                     $_.Attributes -notmatch "System"))})) 
    { 
       get-childitem -lit $dir -recurse -force | measure-object -sum -prop Length -EA "SilentlyContinue"  |
          Select-Object @{Name="Path"; Expr={$dir}}, Count, Sum
    } } | Tee-Object -Variable Totals | 
        Select-Object Path, Count, @{Name="Size($unit)"; Expr={[Math]::Round($_.Sum/"1$unit",$round)}} 
    if($total) {
       $totals = $totals | measure-object -Sum -Prop "Count","Sum"
       Get-Item $path | Select @{Name="Path"; Expr={$_}}, 
                               @{Name="Count"; Expr={$totals[0].Sum}}, 
                               @{Name="Size($unit)"; Expr={[Math]::Round((($totals[1].Sum)/"1$unit"),$round)}}
    }
 }
`

