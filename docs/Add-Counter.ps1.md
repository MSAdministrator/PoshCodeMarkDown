---
Author: billbarry
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5016
Published Date: 2014-03-24t20
Archived Date: 2014-03-30t06
---

# add-counter - 

## Description

function add-counter (adds count noteproperty to pipeline input to keep a running row count for display); sample usage

## Comments



## Usage



## TODO



## function

`add-counter`

## Code

`#
 #
 Function Add-Counter {
     [CmdletBinding()]
     Param(
         [parameter(Mandatory=$true, ValueFromPipeline=$true)] $input,
         [string] $Name='Count'
     )
     BEGIN { $i = 0;}
     PROCESS {
         $i++;
         return Add-Member -InputObject $_ -MemberType NoteProperty -Name $Name -Value $i -PassThru
     }
 }
`

