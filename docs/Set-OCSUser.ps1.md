---
Author: jan egil ring
Publisher: 
Copyright: 
Email: jan.egil.ring@powershell.no
Version: 1.0
Encoding: ascii
License: cc0
PoshCode ID: 1223
Published Date: 
Archived Date: 2010-07-18t14
---

# set-ocsuser - set-ocsuser.ps1

## Description

requires quest ad cmdlets. this oneliner sets active directory user attributes for microsoft office communications server 2007.

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 ###########################################################################"
 #
 #
 #
 #
 #
 #
 ###########################################################################"
 
 $Username="demo.user"
 $Fullname="Demo User"
 $serverpool="pool01.ourdomain.com"
 
 $oa = @{'msRTCSIP-ArchivingEnabled'=0; 'msRTCSIP-FederationEnabled'=$true; 'msRTCSIP-InternetAccessEnabled'=$true; 'msRTCSIP-OptionFlags'=257; 'msRTCSIP-PrimaryHomeServer'=$serverpool; 'msRTCSIP-PrimaryUserAddress'=("sip:$Username@ourdomain.com"); 'msRTCSIP-UserEnabled'=$true }
 
 Set-QADUser $Fullname -oa $oa
`

