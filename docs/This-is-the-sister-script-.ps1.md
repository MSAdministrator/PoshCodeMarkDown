---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 1.0
Encoding: ascii
License: cc0
PoshCode ID: 2688
Published Date: 
Archived Date: 2011-05-23t01
---

#  - 

## Description

this is the sister script to a bigger project. i will have the full script and uses on my blog vnoob.com soon

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 $test=(get-folder testing|get-vm)
 
 #$data and the csv is where all the information lies that this script/s pulls
 $data=import-csv book1.csv
 
 
 $hostcredential=Get-Credential "Host Credentials"
 $guestcredential=Get-Credential "Guest Credentials"
 
 foreach ($line in $data)
 {
 $match=$test|?{$line.guestname -eq $_.name}
 
 	IF($match)
 	{
 	invoke-vmscript -vm $match.name -scripttext '&"C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe" "C:\Power\works2.ps1"' -scripttype "powershell" -hostcredential $hostcredential -guestcredential $guestcredential -confirm
 	}
 
 }
`

