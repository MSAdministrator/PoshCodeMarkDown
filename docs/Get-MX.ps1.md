---
Author: lance robinson
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6472
Published Date: 2016-08-12t16
Archived Date: 2016-08-14t11
---

# get-mx - 

## Description

returns the priority mail server (smtp) to send email directly to the smtp server of a particular domain/email address.  uses netcmdlets (get-dns).

## Comments



## Usage



## TODO



## function

`get-mx`

## Code

`#
 #
 function Get-MX {
 param([string] $domain = $( Throw "Query required in the format domain.com or email@domain.com.") )
 	
 	if ($domain.IndexOf("@")) { $domain = $domain.SubString($domain.IndexOf("@")+1) } 
 	  	
   $mxrecords = @(get-dns $domain -type MX | sort-object -Property PREFERENCE -des)
   
   if ($mxrecords.Length -eq 0) { Throw "No records found." }
   	
   $mxrecords[0].EXCHANGE
 }
`

