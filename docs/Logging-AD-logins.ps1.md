---
Author: alan oakland
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 3452
Published Date: 2012-06-12t14
Archived Date: 2012-06-18t09
---

# logging ad logins - 

## Description

this will allow you to create a log to a file share of ad logins.  this creates two files one based on logins to the computer, and the other is based on the username.

## Comments



## Usage



## TODO



## function

``

## Code

`#
 #
 #* FileName: loginTime.ps1
 #*=============================================================================
 #* Created: [09/25/2009]
 #* Author: Alan Oakland
 #*=============================================================================
 #* Purpose: Create a log of login times.
 #*=============================================================================
 
 #*=============================================================================
 #* REVISION HISTORY
 #*=============================================================================
 
 #*=============================================================================
 
 #*=============================================================================
 #* Variable Configuration
 #*=============================================================================
 $adsobj = [ADSI]"WinNT://$domain/$env:username"
 $logpath = "\\server\share\path"
 #*=============================================================================
 #* FUNCTION LISTINGS
 #*=============================================================================
 
 #*=============================================================================
 #* SCRIPT BODY
 #*=============================================================================
 $global:ipinfo = get-wmiobject -class "Win32_NetworkAdapterConfiguration" `
 | Where{($_.IpEnabled -Match "True") -And ($_.DNSDomain -Match $curDomain)}
 if ($ipinfo)
 {
   $global:ip=$ipinfo.ipaddress[0]
 }
 else
 {
   $global:ip=[system.net.dns]::gethostAddresses($null)[0].ipaddresstostring
   if (!($ip))
   {
     $global:ip="IP could not be resolved"
   }
 }
 
 
 
 $fullName = $adsobj.FullName.value.toString()
 if (!(test-path -path "$logfile"))
 {
   $null = new-item $logfile -Type file
 }
 $logData = "$(get-date -Format M/dd/yyyy) - $(get-date -Format H:mm:ss) - Login  - $ip - $env:username - $env:clientname=$env:computername"
 [Array]$entries = Get-Content $logfile
 $entries += $logData
 if (($entries.length-1) -gt $recordstokeep)
 {
   $entries = $entries[0],$entries[($entries.length-$recordstokeep)..($entries.length-1)]
 }
 $entries | Out-File $logfile
 
 if (!(test-path -path "$logfile"))
 {
   $null = new-item $logfile -Type file
 }
 $logData = "$(get-date -Format yyyy/M/dd) - $(get-date -Format H:mm:ss) - Login  - $ip - $env:username =$fullName"
 [Array]$entries = Get-Content $logfile
 $entries += $logData
 if (($entries.length-1) -gt $recordstokeep)
 {
   $entries = $entries[0],$entries[($entries.length-$recordstokeep)..($entries.length-1)]
 }
 $entries | Out-File $logfile
 #*=============================================================================
 #* END OF SCRIPT
 #*=============================================================================
`

