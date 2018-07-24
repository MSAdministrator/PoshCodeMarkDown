---
Author: drdrewl
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 3162
Published Date: 2014-01-13t11
Archived Date: 2014-11-21t19
---

# event log sox audit - 

## Description

sarbanes�oxley (sox) compliance auditing often requires proof of review of the windows security log and remote connections. this script captures server 2008’s event logging and sends it to a csv for review and/or longterm archiving. the security filter below encompasses the account management, audit policy changes, failed logins, and audit cleared filters. i left them for possible granular reporting in the future.

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 $s = "Server01", "Server02", "Server03"
 foreach($server in $s) {$server
 
 #$computername = Get-Content env:computername
 
 $filter_Security = '<QueryList> <Query Id="0" Path="Security">
 	<Select Path="Security">(*[System[Provider[@Name="Microsoft-Windows-Security-Auditing"] and 
 	(Task = 13824 or Task = 13825 or Task = 13826 or Task = 13827 or Task = 13828 or Task = 13829 or
 	Task = 13568 or Task = 13569 or Task = 13570 or Task = 13571 or Task = 13572) or
 	(Task = 12544 and (band(Keywords,4503599627370496)))]]) or (*[System[Provider[@Name="Microsoft-Windows-Eventlog"] and Task = 104]])
 	</Select></Query></QueryList>'
 
 $filter_AcctManagement  = '<QueryList> <Query Id="0" Path="Security">
 	<Select Path="Security">*[System[Provider[@Name="Microsoft-Windows-Security-Auditing"] and 
 	(Task = 13824 or Task = 13825 or Task = 13826 or Task = 13827 or Task = 13828 or Task = 13829)]]
 	</Select></Query></QueryList>'
 	
 $filter_AuditPolicyChanges  = '<QueryList> <Query Id="0" Path="Security">
 	<Select Path="Security">*[System[Provider[@Name="Microsoft-Windows-Security-Auditing"] and 
 	(Task = 13568 or Task = 13569 or Task = 13570 or Task = 13571 or Task = 13572 or Task = 13573)]]
 	</Select></Query></QueryList>'
 	
 $filter_FailedLogins  = '<QueryList> <Query Id="0" Path="Security">
 	<Select Path="Security">*[System[Provider[@Name="Microsoft-Windows-Security-Auditing"] and 
 	(Task = 12544 and (band(Keywords,4503599627370496)))]]
 	</Select></Query></QueryList>'
 	
 $filter_AuditCleared  = '<QueryList> <Query Id="0" Path="Security">
 	<Select Path="Security">*[System[Provider[@Name="Microsoft-Windows-Eventlog"] and Task = 104]]
 	</Select></Query></QueryList>'
 	
 $filter_RDP  = '<QueryList> <Query Id="0" Path="Microsoft-Windows-TerminalServices-RemoteConnectionManager/Operational">
 	<Select Path="Microsoft-Windows-TerminalServices-RemoteConnectionManager/Operational">*[System[Provider[@Name="Microsoft-Windows-TerminalServices-RemoteConnectionManager"] and (EventID=1149)]]
 	</Select></Query></QueryList>'	
 
 Get-WinEvent -computername $server -FilterXml $filter_RDP | Export-Csv \\networkpath\$server.RDP.csv
 Get-WinEvent -computername $server -FilterXml $filter_Security | Select-Object -Property 'Message','ID','Task','RecordID','LogName','ProcessID','ThreadID','MachineName','TimeCreated','TaskDisplayName' | Export-Csv \\networkpath\$server.Security.csv
`

