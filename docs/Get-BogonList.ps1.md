---
Author: rich kusak
Publisher: 
Copyright: 
Email: 
Version: 1.3.1.0
Encoding: ascii
License: cc0
PoshCode ID: 2913
Published Date: 2011-08-14t07
Archived Date: 2011-11-05t17
---

# get-bogonlist - 

## Description

heres a function to quickly look up the latest version of the bogon list maintained by team cymru from within powershell.

## Comments



## Usage



## TODO



## function

`get-bogonlist`

## Code

`#
 #
 function Get-BogonList {
 <#
 	.SYNOPSIS
 		Gets the bogon list.
 	
 	.DESCRIPTION
 		The Get-BogonList function retrieves the bogon prefix list maintained by Team Cymru.
 		A bogon prefix is a route that should never appear in the Internet routing table.
 		A packet routed over the public Internet (not including over VPNs or other tunnels) should never have a source address in a bogon range.
 		These are commonly found as the source addresses of DDoS attacks.
 		Bogons are defined as Martians (private and reserved addresses defined by RFC 1918 and RFC 5735) and netblocks that have not been allocated
 		to a regional internet registry (RIR) by the Internet Assigned Numbers Authority. Fullbogons are a larger set which also includes IP space
 		that has been allocated to an RIR, but not assigned by that RIR to an actual ISP or other end-user.
 	
 	.PARAMETER Aggregated
 		Retrieves the aggreated traditional bogon list. By default the unaggregated list is retrieved.
 	
 	.PARAMETER FullBogons
 		Retrieves the full bogon list.
 	
 	.PARAMETER IPVersion
 		Specifies which IP version of the full bogon list will be retrieved. The FullBogons parameter is required.
 		The default value is version 4.
 		
 	.INPUTS
 		None
 	
 	.OUTPUTS
 		PSObject
 	
 	.EXAMPLE
 		Get-BogonList
 		Retrieves the traditional unaggregated bogon list from Team Cymru.
 		
 	.EXAMPLE
 		Get-BogonList -Verbose
 		Retrieves the traditional unaggregated bogon list from Team Cymru with additional header information.
 			
 	.EXAMPLE
 		Get-BogonList -Aggregated
 		Retrieves the traditional aggregated bogon list from Team Cymru.
 	
 	.EXAMPLE
 		Get-BogonList -FullBogons
 		Retrieves the full IPv4 bogon list from Team Cymru.
 	
 	.EXAMPLE
 		Get-BogonList -FullBogons -IPVersion 6
 		Retrieves the full IPv6 bogon list from Team Cymru.
 
 	.NOTES
 		Name: Get-BogonList
 		Author: Rich Kusak (rkusak@cbcag.edu)
 		Created: 2010-01-31
 		LastEdit: 2011-08-14 09:26
 		Version: 1.3.1.0
 		
 		
 	.LINK
 		http://www.team-cymru.org/Services/Bogons/
 
 #>
 	
 	[CmdletBinding(DefaultParameterSetName='Traditional')]
 	param (
 		[Parameter(ParameterSetName='Traditional')]
 		[switch]$Aggregated,
 		
 		[Parameter(ParameterSetName='Full')]
 		[switch]$FullBogons,
 		
 		[Parameter(ParameterSetName='Full')]
 		[ValidateSet(4,6)]
 		[int]$IPVersion = 4
 	)
 	
 	$webClient = New-Object System.Net.WebClient
 	
 	if ($psCmdlet.ParameterSetName -eq 'Traditional') {
 		$referencePage = $webClient.DownloadString('http://www.team-cymru.org/Services/Bogons/')
 
 		$traditionalUpdated = [regex]::Match($referencePage, 'Updated:.*(\d{2}\s\w+\s\d{4})', 'IgnoreCase').Groups[1].Value
 		
 		$currentVersion = [regex]::Match($referencePage, 'version:.*(\d+\.\d+)', 'IgnoreCase').Groups[1].Value
 		
 		if ($PSBoundParameters['Verbose']) {
 			Write-Output 'Team Cymru Traditional Bogon List'
 			'Updated: {0}' -f $traditionalUpdated
 			'Current version: {0}' -f $currentVersion
 		}
 		
 		if ($Aggregated) {
 			foreach ($bogon in $webClient.DownloadString('http://www.team-cymru.org/Services/Bogons/bogon-bn-agg.txt') -split "`n") {
 				New-Object PSObject -Property @{'Aggregated' = $bogon}
 			}
 
 		} else {
 			foreach ($bogon in $webClient.DownloadString('http://www.team-cymru.org/Services/Bogons/bogon-bn-nonagg.txt') -split "`n") {
 				New-Object PSObject -Property @{'Unaggregated' = $bogon}
 			}
 		}
 	
 	if ($psCmdlet.ParameterSetName -eq 'Full') {
 		if (-not $FullBogons) {
 			return Write-Error 'The FullBogons parameter is required to set the IPVersion.'
 		}
 		
 		switch ($IPVersion) {
 			4 {
 				$bogons = $webClient.DownloadString('http://www.team-cymru.org/Services/Bogons/fullbogons-ipv4.txt') -split "`n"
 				$propertyName = 'IPv4FullBogons'
 				break
 			}
 			6 {
 				$bogons = $webClient.DownloadString('http://www.team-cymru.org/Services/Bogons/fullbogons-ipv6.txt') -split "`n"
 				$propertyName = 'IPv6FullBogons'
 				break
 			}
 		}
 		
 		$fullUpdated = [regex]::Match($bogons[0], '\(.+\)').Value.Trim('()')
 		
 		if ($PSBoundParameters['Verbose']) {
 			Write-Output 'Team Cymru Full Bogon List'
 			'Updated: {0}' -f $fullUpdated
 		}
 		
 		foreach ($bogon in $bogons -match '^\d') {
 			New-Object PSObject -Property @{$propertyName = $bogon}
 		}
`

