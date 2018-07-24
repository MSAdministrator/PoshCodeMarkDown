---
Author: thomas torggler
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 4196
Published Date: 2013-06-09t12
Archived Date: 2013-11-27t06
---

# exchange perfcounters - 

## Description

a quick script to re-create exchange 2013 performance counters, more information check

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 Add-PSSnapin Microsoft.Exchange.Management.PowerShell.Setup
 Get-ChildItem "$exInstall\Setup\Perf" | Where-Object {$_.Name -match ".xml"} | Foreach {New-PerfCounters -DefinitionFileName $_.FullName}
`

