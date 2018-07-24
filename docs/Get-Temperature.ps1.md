---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 2.5
Encoding: ascii
License: cc0
PoshCode ID: 4211
Published Date: 2013-06-21t04
Archived Date: 2013-06-25t00
---

# get-temperature - 

## Description

reading temperature data from the carbon dioxide information analysis center

## Comments



## Usage



## TODO



## function

`get-temperature`

## Code

`#
 #
 function Get-Temperature {
 [CmdletBinding()]
 param(
 	$maxTemps = "~\Downloads\ushcn2012_tob_tmax.txt",
 	$minTemps = "~\Downloads\ushcn2012_tob_tmin.txt",
 	$StationID = 307167,
 
   $Years = 5
 )
 
   $max = Select-String USH0+$StationID $maxTemps | % line
   $min = Select-String USH0+$StationID $minTemps | % line
 
   if($max.Count -ne $min.Count) {
     Write-Warning "TODO: write code to throw away years until we get data in both files."
   }
 
   $temps = @{}
   foreach($y in (-$Years)..-1) {
     $Year = [int]$max[$y].Substring(12,4)
     $temps.$Year = @{}
     Write-Verbose "$Year $($temps.$Year.GetType().FullName)"
     foreach($Month in 1..12){ 
       Write-Verbose "$Year-$Month"
       $temps.$year.$Month = @{
         "Max" = .1 * [int]$max[$y].Substring((8+($month*9)),5)
         "Min" = .1 * [int]$min[$y].Substring((8+($month*9)),5)
       }
     }
   }
   $temps
 }
`

