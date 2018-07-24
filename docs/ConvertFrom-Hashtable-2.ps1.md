---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 1819
Published Date: 2011-05-03t13
Archived Date: 2016-04-26t12
---

# convertfrom-hashtable 2 - 

## Description

create psobjects from hashtables. supports converting nested hashtables, and joining multiple pipelined hashtables into one object using join-object

## Comments



## Usage



## TODO



## function

`convertfrom-hashtable`

## Code

`#
 #
 [CmdletBinding()]
    PARAM(
       [Parameter(ValueFromPipeline=$true, Mandatory=$true)]
       [HashTable]$hashtable
    ,
       [switch]$Combine
    ,
       [switch]$Recurse
    )
    BEGIN {
       $output = @()
    }
    PROCESS {
       if($recurse) {
          $keys = $hashtable.Keys | ForEach-Object { $_ }
          Write-Verbose "Recursing $($Keys.Count) keys"
          foreach($key in $keys) {
             if($hashtable.$key -is [HashTable]) {
             }
          }
       }
       if($combine) {
          $output += @(New-Object PSObject -Property $hashtable)
          Write-Verbose "Combining Output = $($Output.Count) so far"
       } else {
          New-Object PSObject -Property $hashtable
       }
    }
    END {
       if($combine -and $output.Count -gt 1) {
          Write-Verbose "Combining $($Output.Count) cached outputs"
          $output | Join-Object
       } else {
          $output
       }
    }
 #}
`

