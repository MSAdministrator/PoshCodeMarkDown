---
Author: nathan hartley
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2558
Published Date: 2011-03-14t13
Archived Date: 2011-03-24t17
---

# get-hostentry.ps1 - 

## Description

queries dns to return the host name and associated ip addresses, given either an ip address or a host name via the pipeline or parameter (accepts arrays).

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 param (
 	[parameter(Mandatory=$true, ValueFromPipeline=$true)]
 		[String[]]$HostnameOrIPs
 )
 process {
 	ForEach ($HostnameOrIP in $HostnameOrIPs) {
 		try {
 			$result = [System.Net.Dns]::GetHostEntry($HostnameOrIP)
 			"" | select @{Name='HostName'; Expression={$result.HostName}}, @{Name='AddressList'; Expression={$result.AddressList}}
 		}
 		catch {
 			Write-Warning ("[{0}] Lookup failed: {1}" -f $HostnameOrIP, $_.exception.InnerException.message)
 		}
 	}
 }
`

