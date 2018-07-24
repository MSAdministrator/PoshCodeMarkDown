---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 2.0
Encoding: ascii
License: cc0
PoshCode ID: 81
Published Date: 
Archived Date: 2009-01-06t14
---

# select-random - 

## Description

select a user-defined number of random elements from the collection … which can be passed as a parameter or input via the pipeline. an improvement over http

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 ###
 ###
 ###
 param([int]$count=1, [array]$inputObject=$null) 
 
 BEGIN {
    if ($args -eq '-?') {
 @"
 Usage: Select-Random [[-Count] <int>] [-inputObject] <array> (from pipeline) [-?]
 
 Parameters:
  -Count      : The number of elements to select.
  -Collection : The collection from which to select a random element.
  -?          : Display this usage information and exit
 
 Examples:
  PS> $arr = 1..5; Select-Random $arr
  PS> 1..10 | Select-Random -Count 2
 
 "@
 exit
    } 
    elseif ($inputObject) 
    {
       Write-Output $inputObject | &($MyInvocation.InvocationName) -Count $count
       break;
    }
    else
    {
       $seen = 0
       $selected = new-object object[] $count
       $rand = new-object Random
    }
 }
 PROCESS {
    if($_) {
       $seen++
       if($seen -lt $count) {
          $selected[$seen-1] = $_
       elseif($rand.NextDouble() -gt $count/$seen)
       {
          $selected[$rand.Next(0,$count)] = $_
       }
    }
 }
 END {
    if (-not $inputObject)
       Write-Verbose "Selected $count of $seen elements"
       Write-Output $selected
    }
 }
`

