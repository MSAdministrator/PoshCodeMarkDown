---
Author: rcookiemonster
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5494
Published Date: 2015-10-08t22
Archived Date: 2015-03-23t13
---

# sysmon event data - 

## Description

example extracting data from sysmon event logs.

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
         . "\\path\to\Get-WinEventData.ps1"
 
 
         Get-WinEvent -FilterHashtable @{logname="Microsoft-Windows-Sysmon/Operational";id=3} |
             Get-WinEventData |
             select -first 1 -Property *
 
             <#
 
                 ...
                 EventDataUtcTime             : 10/8/2014 10:41 PM
                 EventDataProcessGuid         : {00000000-A3D1-5435-0000-001094C60700}
                 EventDataProcessId           : 5248
                 EventDataImage               : C:\Program Files (x86)\Plex\Plex Media Server\PlexDlnaServer.exe
                 EventDataUser                : *************\*************
                 EventDataProtocol            : tcp
                 EventDataInitiated           : false
                 EventDataSourceIsIpv6        : false
                 EventDataSourceIp            : 127.0.0.1
                 EventDataSourceHostname      : *************
                 EventDataSourcePort          : 12804
                 EventDataSourcePortName      : 
                 EventDataDestinationIsIpv6   : false
                 EventDataDestinationIp       : 127.0.0.1
                 EventDataDestinationHostname : *************
                 EventDataDestinationPort     : 12805
                 EventDataDestinationPortName : 
                 ...
 
             #>
 
         Get-WinEvent -FilterHashtable @{logname="Microsoft-Windows-Sysmon/Operational";id=3} | get-wineventdata | ?{$_.EventDataImage -like "*plex*"} |
             select EventDataSourceIP, EventDataDestinationIP 
 
             <#
 
                 EventDataSourceIp EventDataDestinationIp
                 ----------------- ----------------------
                 127.0.0.1         127.0.0.1                   
                 127.0.0.1         127.0.0.1             
                 192.168.1.4       192.168.1.4           
                 192.168.1.4       192.168.1.4           
                 127.0.0.1         127.0.0.1             
                 127.0.0.1         127.0.0.1             
                 127.0.0.1         127.0.0.1             
                 127.0.0.1         127.0.0.1             
                 192.168.1.4       192.168.1.115         
                 192.168.1.4       192.168.1.115         
                 192.168.1.4       192.168.1.115         
 
             #>
`

