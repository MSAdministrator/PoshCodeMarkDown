---
Author: will steele
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 3819
Published Date: 2012-12-12t23
Archived Date: 2015-12-17t14
---

# addto-hostsfile - 

## Description

add entries to the windows hosts file.  the function checks first to see if the entry exists in the file.  if the entry does not exist, the function adds the entry and verifies it was added.  please feel free to add improvements.  the function utility.time-stamp is another helper function i use to denote the current time.  it can be found here

## Comments



## Usage



## TODO



## function

`addto-hostsfile`

## Code

`#
 #
 function AddTo-HostsFile{
 
 	<#
 		.DESCRIPTION
 			This function checks to see if an entry exists in the hosts file.
 			If it does not, it attempts to add it and verifies the entry.
 
 		.EXAMPLE
 			Networkign.AddTo-Hosts -IPAddress 192.168.0.1 -HostName MyMachine
 
 		.EXTERNALHELP
 			None.
 
 		.FORWARDHELPTARGETNAME
 			None.
 
 		.INPUTS
 			System.String.
 
 		.LINK
 			None.
 
 		.NOTES
 			None.
 
 		.OUTPUTS
 			System.String.
 
 		.PARAMETER IPAddress
 			A string representing an IP address.
 
 		.PARAMETER HostName
 			A string representing a host name.
 
 		.SYNOPSIS
 			Add entries to the hosts file.
 	#>
 
   param(
     [parameter(Mandatory=$true,position=0)]
 	[string]
 	$IPAddress,
 	[parameter(Mandatory=$true,position=1)]
 	[string]
 	$HostName
   )
 
 	$HostsLocation = "$env:windir\System32\drivers\etc\hosts";
 	$NewHostEntry = "`t$IPAddress`t$HostName";
 
 	if((gc $HostsLocation) -contains $NewHostEntry)
 	{
 	  Write-Host "$(Time-Stamp): The hosts file already contains the entry: $NewHostEntry.  File not updated.";
 	}
 	else
 	{
     Write-Host "$(Time-Stamp): The hosts file does not contain the entry: $NewHostEntry.  Attempting to update.";
 		Add-Content -Path $HostsLocation -Value $NewHostEntry;
 	}
 
 	if((gc $HostsLocation) -contains $NewHostEntry)
 	{
 	  Write-Host "$(Time-Stamp): New entry, $NewHostEntry, added to $HostsLocation.";
 	}
 	else
 	{
     Write-Host "$(Time-Stamp): The new entry, $NewHostEntry, was not added to $HostsLocation.";
 	}
 }
`

