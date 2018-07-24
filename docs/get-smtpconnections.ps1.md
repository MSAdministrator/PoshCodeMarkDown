---
Author: chriskenis
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 3947
Published Date: 2015-02-14t09
Archived Date: 2015-08-03t02
---

# get-smtpconnections - 

## Description

extract unique list of ip addresses from single or mulitple smtp logfiles (w3svc)

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 param (
 $logpath = "C:\WINDOWS\system32\LogFiles\SMTPSVC1"
 $logfiles = @("ex130213.log","ex130214.log")
 $regex = "(?:[0-9]{1,3}\.){3}[0-9]{1,3}"
 )
 $smtphosts = @()
 foreach ($logFile in $logfiles){
 	$logfilepath = Join-Path $logpath $logfile
 	write-host "getting smtp connections from $logfile" -foregroundcolor green
 	$smtphosts += select-string -Path $logfilepath -Pattern $regex -AllMatches | %{ $_.Matches } | % { $_.Value } | sort -Unique
 	}
 $smtphosts | sort -Unique
`

