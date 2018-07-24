---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 1013
Published Date: 
Archived Date: 2009-04-16t01
---

#  - 

## Description

runs a fulltextsqlquery against your local moss farm; useful as a quick(!) search query test workbench.

## Comments



## Usage



## TODO



## function

`run-query`

## Code

`#
 #
 function Run-Query($siteUrl, $queryText)
 {
 	[reflection.assembly]::loadwithpartialname("microsoft.sharePOint") | out-null
 	[reflection.assembly]::loadwithpartialname("microsoft.office.server") | out-null
 	[reflection.assembly]::loadwithpartialname("microsoft.office.server.search") | out-null
 	$s = [microsoft.sharepoint.spsite]$siteUrl
 	$q = new-object microsoft.office.server.search.query.fulltextsqlquery -arg $s
 	$q.querytext = $queryText
 	$q.RowLimit = 100
 	$q.ResultTypes = "RelevantResults"
 	$dt = $q.Execute()
 	$r = $dt["RelevantResults"]
 
 	$output = @()
 	
 	while ($r.Read()) {
 		$o = new-object PSObject
 
 		0..($r.FieldCount-1) | foreach {
 			add-member -inputObject $o -memberType "NoteProperty" -name $r.GetName($_) -value $r[$_].ToString()
 		}
 		
 		
 		$output += $o
 	}
 	
 	return $output
 }
 
`

