---
Author: richard young
Publisher: 
Copyright: 
Email: 
Version: 0.0
Encoding: ascii
License: cc0
PoshCode ID: 6720
Published Date: 2017-02-06t11
Archived Date: 2017-02-12t12
---

# standard deviation - 

## Description

the most accurate way of calculating standard deviation (according to the experts) is a method created by b.p.welford, which is detailed in-depth in donald knuth�s �art of computer programming�.

## Comments



## Usage



## TODO



## function

`get-standarddeviation`

## Code

`#
 #
 function Get-StandardDeviation {
     [CmdletBinding()]
     Param (
     [Parameter(Mandatory=$true, ValueFromPipelineByPropertyName=$true,ValueFromPipeline=$true)]
     [ValidateNotNullOrEmpty()]
     [double[]]$Values
     )
     Begin {
         $count=0.0
         $mean=0.0
         $sum=0.0
  
     Process {
         foreach ($value in $values) {
             ++$count
             $delta = $mean + (($value - $mean) / $count)
             $sum += ($value - $mean) * ($value - $delta)
             $mean = $delta
  
     End {
         $VariancePopulation = $sum/($count)
         $VarianceSample = $sum/($count-1)
         $obj=[PSCustomObject]@{
             "VariancePopulation" = $VariancePopulation
             "VarianceSample" = $VarianceSample
             "STDEVPopulation" = [Math]::Sqrt($VariancePopulation)
             "STDEVSample" = [Math]::Sqrt($VarianceSample)
             "Mean" = $mean
             "Count" = $count
         Write-Output $obj
  
`

