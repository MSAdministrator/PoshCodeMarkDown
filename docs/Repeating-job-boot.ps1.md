---
Author: godeater
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: utf-8
License: cc0
PoshCode ID: 5375
Published Date: 2015-08-20t19
Archived Date: 2015-05-05t01
---

# repeating job @ boot - 

## Description

a snippet which will setup a job that repeats every 5 minutes at boot time.

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 [scriptblock]$UpdateDuckDns = {
     $Encoding = [System.Text.Encoding]::UTF8;
     $duckdns_url = "https://www.duckdns.org/update?domains=YOUR_DOMAIN&token=YOUR_TOKEN&ip=";
 
     $HTTP_Response = Invoke-WebRequest -Uri $duckdns_url;
     $Text_Response = $Encoding.GetString($HTTP_Response.Content);
 
     if($Text_Response -ne 'OK'){
         Write-EventLog �LogName Application �Source "DuckDNS Updater" �EntryType Information �EventID 1 �Message "This is a test message.";
     }
 }
 
 [scriptblock]$SetupRepeatingJob = {
     $RepeatTimeSpan = New-TimeSpan -Minutes 5;
     $DurationTimeSpan = New-TimeSpan -Days 1000;
     $At = $(Get-Date) + $RepeatTimeSpan;
     $UpdateTrigger = New-ScheduledTaskTrigger -Once -At $At -RepetitionInterval $TimeSpan -RepetitionDuration $DurationTimeSpan;
     Register-ScheduledJob -Name RunDuckDnsUpdate -ScriptBlock $UpdateDuckDns -Trigger $UpdateTrigger;
 }
 
 New-EventLog  �LogName Application �Source "DuckDNS Updater"
 $StartTrigger = New-JobTrigger -AtStartup
 Register-ScheduledJob -Name StartDuckDnsJob -ScriptBlock $SetupRepeatingJob -Trigger $StartTrigger
`

