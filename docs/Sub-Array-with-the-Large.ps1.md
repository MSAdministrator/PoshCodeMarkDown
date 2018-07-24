---
Author: bobloblaw
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 4177
Published Date: 2014-05-21t08
Archived Date: 2014-11-13t07
---

# sub-array with the large - 

## Description

http

## Comments



## Usage



## TODO



## function

`new-randomarray`

## Code

`#
 #
 
 function New-RandomArray{
     #.SYNOPSIS
 
     #.DESCRIPTION
 
     [cmdletbinding()]
     Param(
 
         [Alias('ArrayLength','Count','Size')]
         [int]$Length=300,
 
         [int]$Maximum=1000,
         
         [int]$Minimum=-1000
     )
 
     Write-Verbose 'Generating random array ...'
     return (1..$Length | ForEach-Object {Get-Random -Maximum $Maximum -Minimum $Minimum})
 }
 
 function Get-MaxSumSubArray{
     #.SYNOPSIS
     
     #.DESCRIPTION
 
     [cmdletbinding()]
     Param(
         [int[]]$Array=(New-RandomArray -Verbose),
 
         [int]$SubArrayLength=10
     )
     
     if($SubArrayLength -ge $Array.Count){
         throw "You are not finding a sub-array."
     }
 
     $BeginningIndexOfLastSubArray = $Array.Count - $SubArrayLength
     
     0..$BeginningIndexOfLastSubArray |
     ForEach-Object {$array[$_..($_+$SubArrayLength-1)] | 
     Measure-Object -Sum} | 
     Sort-Object -Property Sum -Descending | 
     Select-Object -First 1
     #>
 
     $MaxSumSubArray = [PSCustomObject]@{Sum=[int]::MinValue;StartIndex=$null;EndIndex=$null}
 
     for($i=0;$i-le$BeginningIndexOfLastSubArray;$i++){
         Write-Verbose "Calculating sub-array {$($array[$i..($i+$SubArrayLength-1)] -join ', ')}."
         $tmpsum = $array[$i..($i+$SubArrayLength-1)] | Measure-Object -Sum
         if($tmpsum.Sum -gt $MaxSumSubArray.Sum){
             $MaxSumSubArray.Sum = $tmpsum.Sum
             $MaxSumSubArray.StartIndex = $i
             $MaxSumSubArray.EndIndex = $i+$SubArrayLength-1
         }
     }
 
     return $MaxSumSubArray
 }
 
 Get-MaxSumSubArray -Verbose
`

