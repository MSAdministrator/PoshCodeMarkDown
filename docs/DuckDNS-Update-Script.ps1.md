---
Author: godeater
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5378
Published Date: 2016-08-21t09
Archived Date: 2016-11-08t06
---

# duckdns update script - 

## Description

a pure powershell script to run periodic updates to your duckdns ip address.

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 #########################################################################
 #########################################################################
 
 [string]$MyDomain       = "YOURDOMAIN"
 [string]$MyToken        = "YOURTOKEN"
 [int]$MyUpdateInterval  = 5
 
 [string]$MyIP     = ""
 
 
 [scriptblock]$UpdateDuckDns = {
     param(
         [Parameter(Mandatory=$true,Position=0)]
         [string]$strUrl
     )
     $Encoding = [System.Text.Encoding]::UTF8;
 
     $HTTP_Response = Invoke-WebRequest -Uri $strUrl;
 
     $Text_Response = $Encoding.GetString($HTTP_Response.Content);
 
     if($Text_Response -ne "OK"){
         Write-EventLog -LogName Application -Source "DuckDNS Updater" -EntryType Information -EventID 1 -Message "DuckDNS Update failed for some reason. Check your Domain or Token.";
     }
 }
 
 [scriptblock]$SetupRepeatingJob = {
     param(
         [Parameter(Mandatory=$true,Position=0)]
         [string]$strDomain,
         [Parameter(Mandatory=$true,Position=1)]
         [string]$strToken,
         [Parameter(Mandatory=$true,Position=2)]
         [int]$iUpdateInterval,
         [Parameter(Mandatory=$false,Position=3)]
         [string]$strIP=""
     )
     $duckdns_url = "https://www.duckdns.org/update?domains=" + $strDomain + "&token=" + $strToken + "&ip=" + $strIP;
 
     $RepeatTimeSpan = New-TimeSpan -Minutes $iUpdateInterval;
 
     $At = $(Get-Date) + $RepeatTimeSpan;
 
     $UpdateTrigger = New-JobTrigger -Once -At $At -RepetitionInterval $RepeatTimeSpan -RepeatIndefinitely;
 
     Register-ScheduledJob -Name "RunDuckDnsUpdate" -ScriptBlock $UpdateDuckDns -Trigger $UpdateTrigger -ArgumentList @($duckdns_url);
 }
 
 if(!([Security.Principal.WindowsPrincipal] [Security.Principal.WindowsIdentity]::GetCurrent()).IsInRole([Security.Principal.WindowsBuiltInRole] "Administrator")){
     Write-Warning "You need to run this from an Administrator Powershell prompt"
     Break
 }
 
 if (!([System.Diagnostics.EventLog]::SourceExists("DuckDNS Updater"))){
     New-EventLog  -LogName "Application" -Source "DuckDNS Updater"
 }
 
 $StartTrigger = New-JobTrigger -AtStartup
 
 if($MyIP.Length -ne 0){
     Register-ScheduledJob -Name "StartDuckDnsJob" -ScriptBlock $SetupRepeatingJob -Trigger $StartTrigger -ArgumentList @($MyDomain,$MyToken,$MyUpdateInterval,$MyIP)
     & $SetupRepeatingJob $MyDomain $MyToken $MyUpdateInterval $MyIP
 } else {
     Register-ScheduledJob -Name "StartDuckDnsJob" -ScriptBlock $SetupRepeatingJob -Trigger $StartTrigger -ArgumentList @($MyDomain,$MyToken,$MyUpdateInterval)
     & $SetupRepeatingJob $MyDomain $MyToken $MyUpdateInterval
 }
 
 Write-Host "All done - your DuckDNS will now update automatically, and will continue to do so across system restarts."
 Write-Host "Have a nice day!"
`

