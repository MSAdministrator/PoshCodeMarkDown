---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 70
Published Date: 
Archived Date: 2008-11-20t11
---

# select-alive - 

## Description

use as a filter to select computers from a stream on which to act upon later in the pipeline. example

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 filter Select-Alive {
 	param ( [switch]$Verbose )
 	trap {
 		Write-Verbose "$(get-date -f 's') ping failed: $computer"
 		continue
 	}
 	if ($Verbose) {
 		$VerbosePreference = "continue"
 		$ErrorActionPreference = "continue"
 	}
 	else {
 		$VerbosePreference = "silentlycontinue"
 		$ErrorActionPreference = "silentlycontinue"
 	}
 	Write-Verbose "$(get-date -f 's') ping start"
 	$ping = New-Object System.Net.NetworkInformation.Ping
 	$reply = $null
 	$_ | foreach-object {
 		$obj = $_
 		switch ($obj.psbase.gettype().name) {
 			"DirectoryEntry"    { $cn = $obj.dnshostname[0] }
 			"IPHostEntry"		{ $cn = $obj.HostName }
 			"PSCustomObject"    { $cn = $obj.Name }
 			"SearchResult"      { $cn = $obj.properties['dnshostname'][0] }
 			"String"            { $cn = $obj.trim() }
 		}
 		Write-Verbose "$(get-date -f 's') pinging $cn..."
 		$searchCount++
 		$reply = $ping.Send($cn)
 		if ($reply.status -eq "Success") {
 			$cn; $pingCount++
 		}
 	}
 	Write-Verbose "$(get-date -f 's') ping end - $pingCount/$searchCount online"
 }
`

