---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 1718
Published Date: 
Archived Date: 2010-04-11t06
---

# select-expand - 

## Description

like select-object -expand, but with recursive iteration of a select chain

## Comments



## Usage



## TODO



## function

`select-expand`

## Code

`#
 #
 function Select-Expand {
 .Synopsis
    Like Select-Object -Expand, but with recursive iteration of a select chain
 .Description
    Takes a dot-separated series of properties to expand, and recursively iterates the output of each property ...
 .Parameter Property
    A collection of property names to expand.
    
    Each property can be a dot-separated series of properties, and each one is expanded, iterated, and then evaluated against the next
 .Parameter InputObject
    The input to be selected from
 .Parameter Unique
    If set, this becomes a pipeline hold-up, and the total output is checked to ensure uniqueness
 .Parameter EmptyToo
    If set, Select-Expand will include empty/null values in it's output
 .Example
    Get-Help Get-Command | Select-Expand relatedLinks.navigationLink.uri -Unique
 
    This will return the online-help link for Get-Command.  It's the equivalent of running the following command:
 
    C:\PS> Get-Help Get-Command | Select-Object -Expand relatedLinks | Select-Object -Expand navigationLink | Select-Object -Expand uri | Where-Object {$_} | Select-Object -Unique
 #>
 param(
    [Parameter(ValueFromPipeline=$false,Position=0)]
    [string[]]$Property
 ,
    [Parameter(ValueFromPipeline=$true)]
    [Alias("IO")]
    [PSObject[]]$InputObject
 ,
    [Switch]$Unique
 ,
    [Switch]$EmptyToo
 )
 begin { 
    if($unique) {
       $output = @()
    }
 }
 process {
    foreach($io in $InputObject) {
       foreach($prop in $Property -split "\.") {
          if($io -ne $null) {
             $io = $io | Select-Object -Expand $prop
             Write-Verbose $($io | out-string)
          }
       }
       if(!$EmptyToo -and ($io -ne $null)) {
          $io = $io | Where-Object {$_}
       }
       if($unique) {
          $output += @($io)
       } else {
          Write-Output $io
       }
    }
 }
 end {
    if($unique) {
       Write-Output $output | Select-Object -Unique
    }
 }
 }
 
 New-Alias slep Select-Expand
`

