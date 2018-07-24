---
Author: aleksandar
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 1079
Published Date: 2010-05-04t14
Archived Date: 2016-03-06t01
---

# get-wminamespace - 

## Description

in order to enumerate all the wmi namespaces, you must first connect to the “root” namespace, query for all the “__namespace” instances, and for each instance recursively repeat this process. you can use the computername parameter of get-wminamespace to list the wmi namespaces on the remote computer.

## Comments



## Usage



## TODO



## function

`get-wminamespace`

## Code

`#
 #
 
 function Get-WmiNamespace {
 	param (
 	[string]$rootns = "root",
 	[string]$computerName = ".",
 	$credential
 	)
 
 	if ($credential -is [String] ) {
 		$credential = Get-Credential $credential
 	}
 
 	if ($credential -eq $null) {
 		gwmi -class __namespace -namespace $rootns -computerName $computerName |
 		where {$_.name} | foreach {
 			$ns = "{0}\{1}" -f $rootns,$_.name
 			$ns
 			Get-WmiNamespace -rootns $ns -computer $computerName
 		}
 	}
 	else {
 		gwmi -class __namespace -namespace $rootns -computerName $computerName -credential $credential |
 		where {$_.name} | foreach {
 			$ns = "{0}\{1}" -f $rootns,$_.name
 			$ns
 			Get-WmiNamespace -rootns $ns -computer $computerName -credential $credential
 		}
 	}
 }
`

