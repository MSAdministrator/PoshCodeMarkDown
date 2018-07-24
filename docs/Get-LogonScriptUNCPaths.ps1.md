---
Author: justin grote
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6040
Published Date: 2016-10-05t22
Archived Date: 2016-05-17t12
---

# get-logonscriptuncpaths - 

## Description

#this script extracts the unc paths from all login scripts associated to users. can also be used to search any folder or files for unc paths by tweaking the netlogonpath and netlogonfilestosearch arguments.

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 
 
 param (
 [String]$netlogonpath = "\\" + $env:USERDNSDomain + "\netlogon\",
 [Switch]$IncludeAllNetlogonBatVBS = $true,
 $NetLogonFilesToSearch = ("*.bat","*.vbs")
 )
 
 
 $logonscripts = get-aduser -filter * -properties scriptpath | select samaccountname,scriptpath
 
 $scriptpaths = $logonscripts.scriptpath | foreach {if ($PSItem) {$PSItem.tolower()}}
 
 if ($NetLogonFilesToSearch) {
     $scriptpaths += (get-childitem $netlogonpath -include $NetLogonFilesToSearch -recurse).fullname | foreach {if ($PSItem) {$PSItem.tolower()}}
 }
 
 $scriptpaths = $scriptpaths.trim() | select -unique | sort
 
 foreach ($scriptpath in $scriptpaths) {
     write-progress "Scanning $scriptpath"
     if ($scriptpath -notmatch '\\') {$scriptpath = $netlogonpath + $scriptpath}
 
     select-string -Path $scriptpath -allmatches -pattern '\\\\.*\\.*' -erroraction stop | select filename,linenumber,line
 }
`

