---
Author: david
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5902
Published Date: 2016-06-19t22
Archived Date: 2016-09-09t00
---

# check for service outage - 

## Description

this script checks for any service that is not running that needs to be running. this script creates a temp file with a list of the services that are not running and when this is detected as having values the script imports the entries from the .csv and emails them to the appropriate individuals. this script was designed to be ran repeatedly as a scheduled task.

## Comments



## Usage



## TODO



## function

``

## Code

`#
 #
 function RunSendEmail
 {
 	$date = Get-Date -Format F
     
     $NewBody" -SmtpServer EMAILSERVER
 }
 
 Get-WmiObject Win32_Service | Where-Object {$_.Name -like '*Backup*' -and $_.StartMode -eq 'Auto' -and $_.State -eq 'Stopped'} | Select-Object Name | export-csv C:\temp\services.csv -NoTypeInformation
 Get-WmiObject Win32_Service | Where-Object {$_.Name -like '*DLO*' -and $_.StartMode -eq 'Auto' -and $_.State -eq 'Stopped'} | Select-Object Name | export-csv C:\temp\services.csv -NoTypeInformation -Append
 Get-WmiObject Win32_Service | Where-Object {$_.Name -like '*BKUPEXEC*' -and $_.StartMode -eq 'Auto' -and $_.State -eq 'Stopped'} | Select-Object Name | export-csv C:\temp\services.csv -NoTypeInformation -Append
 
 $NewBody = "The following list are services found that are listed as to be automatically running`f"
 $NewBody = $NewBody + "and are currently stopped / not running.`f`f"
 $NewBody = $NewBody + "Failed Services:`f"
 $test = Import-CSV C:\temp\services.csv
 if ($test -ne $NULL)
 {
 	Foreach ($line in $test)
 	{
 		$NewLine = $line.Name
 		$NewLine = $NewLine + "`f"
 		$NewBody = $NewBody + $NewLine
 
 	}
 	RunSendEmail
 }
`

