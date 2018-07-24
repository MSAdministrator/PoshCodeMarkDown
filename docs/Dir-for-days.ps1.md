---
Author: dirdays
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 3856
Published Date: 2013-01-02t02
Archived Date: 2013-01-09t07
---

# dir for days - 

## Description

correction of

## Comments



## Usage



## TODO



## function

`create-datepaths`

## Code

`#
 #
 Function Create-DatePaths {
     Param (
         [Parameter(Position=0,Mandatory=$True)]
         [DateTime] $Start,
         [Parameter(Position=1,Mandatory=$True)]
         [ValidateScript({$_ -gt $Start})]
         [DateTime] $End,
         $DestinationPath=(Join-Path $env:UserProfile "Test")
     )
 
     0..(New-TimeSpan $Start $End).Days | % {
         $TargetPath = Join-Path $DestinationPath $(Get-Date $Start -Format "yyyy\\MM-MMMM\\yyyy-MM-dd")
         If (!(Test-Path $TargetPath)) { New-Item $TargetPath -ItemType Directory }
         $Start = $Start.AddDays(1)
     }
 }
`

