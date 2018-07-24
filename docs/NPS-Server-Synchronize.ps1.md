---
Author: ksimon
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6619
Published Date: 2017-11-08t13
Archived Date: 2017-03-15t18
---

# nps server synchronize - 

## Description

this script can be run on a secondary network policy server and will mirror the configuration from the specified primary server, simplifying the management of a redundant or distributed configuration. this script is designed to run as a scheduled task.

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 
 $NPSMaster = "Server1"
 $NPSConfigTempFile = "\\server2\C$\Temp\NPSConfigTemp\NPSConfig-$NPSMaster.xml"
 
 
 if (!(get-eventlog -logname "System" -source "NPS-Sync")) {new-eventlog -logname "System" -source "NPS-Sync"}
 
 trap {write-eventlog -logname "System" -eventID 1 -source "NPS-Sync" -EntryType "Error" -Message "An Error occured during NPS Sync: $_. Script run from $($MyInvocation.MyCommand.Definition)"; exit}
 
 $configExportResult = invoke-command -ComputerName $NPSMaster -ArgumentList $NPSConfigTempFile -scriptBlock {param ($NPSConfigTempFile) netsh nps export filename = $NPSConfigTempFile exportPSK = yes}
 
 $NPSConfigTest = Get-Item $NPSConfigTempFile
 
 $configClearResult = netsh nps reset config
 $configImportResult = netsh nps import filename = $NPSConfigTempFile
 
 remove-item -path $NPSConfigTempFile
 
 $successText = "Network Policy Server Configuration successfully synchronized from $NPSMaster.
 
 Export Results: $configExportResult
 
 Import Results: $configImportResult
 
 Script was run from $($MyInvocation.MyCommand.Definition)"
 
 write-eventlog -logname "System" -eventID 1 -source "NPS-Sync" -EntryType "Information" -Message $successText
`

