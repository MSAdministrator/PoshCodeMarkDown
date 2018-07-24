---
Author: adrianwoodrup
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6657
Published Date: 2016-12-22t14
Archived Date: 2016-12-30t03
---

# stop service and wait... - 

## Description

very simple script that stops a service and waits for it to stop before rebooting. this script can be edited so it runs on every reboot or can be run manually to reboot.

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 This script stops the service, then waits for the service to stop before continuing with the reboot/shutdown 
 The scritp can be pushed to a server/Pc using Group Policy or Registry or run manually.
 The shutdown script Registry key is:
 	HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Group Policy\State\Machine\Scripts\Shutdown\
 
 #>
 $ServiceName = 'Service Name'
 
 Stop-Service $ServiceName
 write-host $ServiceName'...' -NoNewLine
 $TestService = Get-Service  $Service | Select-Object 'Status'
 While($TestService | where {$_.Status -eq 'Running'}){	
 	Write-Host '.'-NoNewLine 
 	Sleep 2	
 	}
 	
`

