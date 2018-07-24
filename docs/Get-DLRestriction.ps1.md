---
Author: dmitry sotnikov
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 4847
Published Date: 2014-01-27t20
Archived Date: 2017-04-21t22
---

# get-dlrestriction - 

## Description

uses qad cmdlets to retrieve distribution list restriction attributes and then provides a list of users which can send email messages to the group.

## Comments



## Usage

get-dlrestriction “worldwide everyone”

## TODO



## function

`get-dlrestriction`

## Code

`#
 #
 ###########################################
 #
 #
 #
 ############################################
 
 function Get-DLRestriction {
 	param([System.String]	$DLName	)
 
   "Checking restrictions for $DLName"
 
   $DL = Get-QADGroup $DLName `
       -IncludedProperties AuthOrig, UnauthOrig, dLMemRejectPerms,`
                       dLMemSubmitPerms, msExchRequireAuthToSendTo
 
   $restricted = $false
 
   if ( $DL -ne $null ) { 
     
     if ( $DL.AuthOrig -ne $null ) { 
       $restricted = $true
       "`nThe following users can send messages to this list:"
       $DL.AuthOrig | Get-QADUser
     }
     
     if ( $DL.UnauthOrig -ne $null ) { 
       $restricted = $true
       "`nAnyone BUT the following users can send messages to this list:"
       $DL.UnauthOrig | Get-QADUser
     }
     
     if ( $DL.dLMemSubmitPerms -ne $null ) { 
       $restricted = $true
       "`nMembers of this group can send messages to this list: $($DL.dLMemSubmitPerms | Get-QADGroup)) :"
       Get-QADGroupMember $DL.dLMemSubmitPerms
     }
     
     if ( $DL.dLMemRejectPerms -ne $null ) { 
       $restricted = $true
       "`nAnyone BUT members of this group can send messages to this list: $($DL.dLMemRejectPerms | Get-QADGroup)) :"
       Get-QADGroupMember $DL.dLMemRejectPerms
     }
     
     if ( $DL.msExchRequireAuthToSendTo ) { 
       $restricted = $true
       "`nOnly authenticated users can send messages to this list.`nExternal senders get blocked."
     }
     
     if ( -not $restricted ) {
       "`nThis list is not restricted. Anyone can email it."
     }
   } else {
     "`nDL $DLName not found."
   }
 }
`

