---
Author: lance robinson
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 1125
Published Date: 
Archived Date: 2009-05-27t16
---

# get-srvpriority - 

## Description

returns the priority srv hostname and port for a particular service and domain.  requires netcmdlets (get-dns).

## Comments



## Usage



## TODO



## function

`get-srvpriority`

## Code

`#
 #
 function Get-SRVPriority {
 param( [string] $query = $( Throw "Query required in the format _Service._Proto.DomainName (ie, `"_ftp._tcp.myserver.net`" or `"_xmpp-client._tcp.gmail.com`").") )
 	
   $srvrecords = @(get-dns $query -type SRV | sort-object -Property PRIORITY -des)
   if ($srvrecords.Length -eq 0) { Throw "No records found." }
   $srvrecords = @($srvrecords | ?{$_.PRIORITY -eq $srvrecords[0].PRIORITY} | sort -Property WEIGHT)
   "$($srvrecords[0].TARGET):$($srvrecords[0].PORT)"
 }
`

