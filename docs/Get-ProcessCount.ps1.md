---
Author: bas bossink
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 573
Published Date: 
Archived Date: 2008-09-15t21
---

# get-processcount - 

## Description

get-processcount returns the number of running processes on local or remote machine. if it can’t find the requested process, it tries to guess what you want.

## Comments



## Usage



## TODO



## function

`get-processcount`

## Code

`#
 #
 Function Get-ProcessCount([string]$process, [string]$computer = "localhost", [switch]$guess) {
 	if($guess) {
 		$clause = [string]::Format("like '%{0}%'",$process)
 	}
 	else {
 		$clause = [string]::Format("='{0}'",$process)
 	}
 	$selectstring = [string]::Format("select * from Win32_Process where name {0}",
 		$clause)
 	$result = get-wmiobject -query $selectstring -ComputerName $computer
 	if($result) { $result | Group-Object Name }
 	else { Write "Process $process could not be found" }
 }
`

