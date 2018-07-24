---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 2.1
Encoding: ascii
License: cc0
PoshCode ID: 1654
Published Date: 2010-02-23t15
Archived Date: 2016-07-10t18
---

# join-collections - 

## Description

performs an inner join on two collections of objects which share a common key value. now works on datatable objects (ie

## Comments



## Usage



## TODO



## script

`join-collections`

## Code

`#
 #
 
 ####################################################################################################
 
 #.Note
 #.Synopsis
 #.Description
 #.Parameter GroupOnColumn
 #.Parameter FirstCollection
 #.Parameter FirstJoinColumn
 #.Parameter SecondCollection
 #.Parameter SecondJoinColumn
 #.Example
 #
 #
 #.Example
 #"@.Split("`n") | ConvertFrom-Csv                               
 #
 #"@.Split("`n") | ConvertFrom-Csv                               
 #
 #
 #.Notes
 
 PARAM(
    $FirstCollection
 ,  [string]$FirstJoinColumn
 ,  $SecondCollection
 ,  [string]$SecondJoinColumn=$FirstJoinColumn
 )
 PROCESS {
    $ErrorActionPreference = "Inquire"
    $JoinHashTable = @{}
    $SecondCollection|%{$JoinHashTable."$SecondJoinColumn" = $_}	
    foreach($first in $FirstCollection) {
       $JoinHashTable."$FirstJoinColumn" | Join-Object $first
    }
 }
 BEGIN {
    function Join-Object {
    Param(
       [Parameter(Position=0)]
       $First
    ,
       [Parameter(ValueFromPipeline=$true)]
       $Second
    )
    BEGIN {
       [string[]] $p1 = $First | gm -type Properties | select -expand Name
    }
    Process {
       $Output = $First | Select $p1
       foreach($p in $Second | gm -type Properties | Where { $p1 -notcontains $_.Name } | select -expand Name) {
          Add-Member -in $Output -type NoteProperty -name $p -value $Second."$p"
       }
       $Output
    }
    }
 }
 #}
`

