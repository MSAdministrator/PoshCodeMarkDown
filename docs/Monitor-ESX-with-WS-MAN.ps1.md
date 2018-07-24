---
Author: carter shanklin
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 928
Published Date: 
Archived Date: 2011-02-23t16
---

# monitor esx with ws-man - 

## Description

this script shows how to monitor esx with powershell v2 ctp3’s new wsman cmdlets.

## Comments



## Usage



## TODO



## function

`get-vmhostwsmaninstance`

## Code

`#
 #
 function Get-VMHostWSManInstance {
 	param (
 	[Parameter(Mandatory=$TRUE,HelpMessage="VMHosts to probe")]
 	[VMware.VimAutomation.Client20.VMHostImpl[]]
 	$VMHost,
 
 	[Parameter(Mandatory=$TRUE,HelpMessage="Class Name")]
 	[string]
 	$class,
 
 	[switch]
 	$ignoreCertFailures,
 
 	[System.Management.Automation.PSCredential]
 	$credential=$null
 	)
 
 	$omcBase = "http://schema.omc-project.org/wbem/wscim/1/cim-schema/2/"
 	$dmtfBase = "http://schemas.dmtf.org/wbem/wscim/1/cim-schema/2/"
 	$vmwareBase = "http://schemas.vmware.com/wbem/wscim/1/cim-schema/2/"
 
 	if ($ignoreCertFailures) {
 		$option = New-WSManSessionOption -SkipCACheck -SkipCNCheck -SkipRevocationCheck
 	} else {
 		$option = New-WSManSessionOption
 	}
 	foreach ($H in $VMHost) {
 		if ($credential -eq $null) {
 			$hView = $H | Get-View -property Value
 			$ticket = $hView.AcquireCimServicesTicket()
 			$password = convertto-securestring $ticket.SessionId -asplaintext -force
 			$credential = new-object -typename System.Management.Automation.PSCredential -argumentlist $ticket.SessionId, $password
 		}
 		$uri = "https`://" + $h.Name + "/wsman"
 		if ($class -cmatch "^CIM") {
 			$baseUrl = $dmtfBase
 		} elseif ($class -cmatch "^OMC") {
 			$baseUrl = $omcBase
 		} elseif ($class -cmatch "^VMware") {
 			$baseUrl = $vmwareBase
 		} else {
 			throw "Unrecognized class"
 		}
 		Get-WSManInstance -Authentication basic -ConnectionURI $uri -Credential $credential -Enumerate -Port 443 -UseSSL -SessionOption $option -ResourceURI "$baseUrl/$class"
 	}
 }
 
`

