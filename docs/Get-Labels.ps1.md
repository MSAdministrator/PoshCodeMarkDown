---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 4144
Published Date: 2013-05-02t22
Archived Date: 2013-05-07t07
---

# get-labels - 

## Description

pulls label-value pairs from text. note that this version is still really optimistic, and assumes that your label-value pairs are each, always, on their own line … but it exposes the get-captures separately so you can write any regex you like (with named captures).

## Comments



## Usage



## TODO



## function

`get-captures`

## Code

`#
 #
 function Get-Captures {
   #.Synopsis
 param( 
   [string]$text,
   [regex]$re,
   [switch]$NoGroup
 )
   end {
     $matches = $re.Matches($data)
     $names = $re.GetGroupNames() | Where { $_ -ne 0 }
     $result = @{}
     foreach($match in $matches | where Success) {
       foreach($name in $names) {
         if($match.Groups[$name].Value) {
           if($NoGroup -or $result.ContainsKey($name)) {
             Write-Output $result
             $result = @{}
           }
           $result.$name = $match.Groups[$name].Value
         }
       }
     }
   }
 }
 
 function Get-Label {
   #.Synopsis
   #.Example
   param(
     [Parameter(Mandatory=$true, ValueFromPipeline=$true)]
     [AllowEmptyString()]
     [string]$data,
 
     [Parameter(ValueFromRemainingArguments=$true, Position = 1)]
     [string[]]$labels = ("Serial Number:","Revocation Date:"),
 
     [switch]$NoGroup,
 
     [switch]$AsObjects
   )
   begin {
     [string[]]$FullData = $data
   }
   process {
     [string[]]$FullData += $data
   }
 
   end {
     $data = $FullData -join "`n"
 
     $names = $labels -replace "\s+" -replace "\W"
 
     $re = "(?m)" + (@(
       for($l=0; $l -lt $labels.Count; $l++) {
         $label = $labels[$l]
         $name = $names[$l]
         "$label\s*(?<$name>.*)\s*`$"
       }) -join "|")
 
     write-verbose $re
 
     if($AsObjects) {
       foreach($hash in Get-Captures $data $re -NoGroup:$NoGroup) {
         New-Object PSObject -Property $hash
       }
     } else {
       Get-Captures $data $re -NoGroup:$NoGroup
     }
   }
 }
`

