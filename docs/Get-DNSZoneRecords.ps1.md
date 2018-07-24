---
Author: saehrig, steven (trac3r726)
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 642
Published Date: 
Archived Date: 2008-10-27t18
---

# get-dnszonerecords - get-dnszonerecords.ps1

## Description

this script was written to query remote dns servers for a record in the local domain. i wrote this in ctp 2 and unsure if it will work in posh v1.

## Comments



## Usage



## TODO



## class

``

## Code

`#
 #
 #==========================================================================
 #
 #
 #==========================================================================
 
 
 $cred = Get-Credential
 $computer = read-inputbox "Enter Server ip"
 $zonename = Get-WmiObject Win32_computersystem -computerName $computer -credential $cred
 $dnszonename = $zonename.domain
 
 get-wmiobject -namespace "root\microsoftdns" -class microsoftdns_atype -computername $computer -credential $cred -filter "containername='$dnszonename'" | ft  dnsservername, ownername, recorddata, ttl
`

