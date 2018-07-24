---
Author: justin grote
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6039
Published Date: 2016-10-05t22
Archived Date: 2016-05-17t07
---

# get-gpouncpaths - 

## Description

this script retrieves the unc paths from all group policies in the domain. useful during file server cutovers and renaings.

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 
 $gpos = get-gpo -all
 foreach ($gpo in $gpos) {
 
     [string]$xmlReport = $gpo | Get-GPOReport -reporttype xml
 
     $searchResults = $xmlReport -split '["\n\r"|"\r\n"|\n|\r]' | where {$_} | Select-String -pattern '\\\\.*\\.*' -context 3
 
     foreach ($searchResult in $searchResults) {
         
         $returnParams = [ordered]@{}
         $returnParams.GUID = $gpo.id
         $returnParams.Name = $gpo.DisplayName
         $returnParams.Line = $searchResult.LineNumber
         $returnParams.Match = $searchResult.Line
         $returnParams.Context = $searchResult.ToString()
 
         new-object PSObject -Property $returnParams
 
     }
 }
`

