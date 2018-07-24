---
Author: thomas torggler
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: utf-8
License: cc0
PoshCode ID: 3508
Published Date: 2015-07-09t11
Archived Date: 2015-05-04t21
---

# set active sync deviceid - 

## Description

get deviceid of all activesync devices for all mailboxes with an activesync partnership, then set the mailboxes activesyncalloweddeviceids to the retrieved deviceid.

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 $s = New-PSSession -ConfigurationName Microsoft.Exchange -Name ExchMgmt -ConnectionUri http://ex14.domain.local/PowerShell/ -Authentication Kerberos
 Import-PSSession $s
 
 $Mailboxes = Get-CASMailbox -Filter {HasActivesyncDevicePartnership -eq $true} -ResultSize Unlimited
 
 $EASMailboxes = $Mailboxes | Select-Object PrimarySmtpAddress,@{N='DeviceID';E={Get-ActiveSyncDeviceStatistics -Mailbox $_.Identity | Select-Object �ExpandProperty DeviceID}}
 
 $EASMailboxes | foreach {Set-CASMailbox $_.PrimarySmtpAddress -ActiveSyncAllowedDeviceIDs $_.DeviceID}
`

