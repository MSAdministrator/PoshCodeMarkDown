---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 1031
Published Date: 2010-04-15t12
Archived Date: 2016-03-05t11
---

# measure-total - 

## Description

a general-purpose function for generating a “totals” row for data with numeric properties.

## Comments



## Usage



## TODO



## script

`measure-total`

## Code

`#
 #
 #.Synopsis
 #.Description
 #.Parameter Property
 #.Example
 #
 #.Example
 #
 
 function measure-total {
 PARAM([string[]]$property)
 END {
    if(!$property) {
       $input.reset()
       $property = $input | gm -type Properties | 
          where { trap{continue} ( new-object "$(@($_.Definition -split " ")[0])" ) -as [double] -ne $null } |
          foreach { $_.Name }
       $input.reset()
    }
    $input.reset()
    $sum = $input | measure $property -sum -erroraction 0
    $output = new-object PSObject
    $sum | % { Add-Member -input $output -Type NoteProperty -Name $_.Property -Value $_.Sum }
    
 }}
 new-alias total measure-total
`

