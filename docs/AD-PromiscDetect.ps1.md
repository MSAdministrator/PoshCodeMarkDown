---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 1.1
Encoding: ascii
License: cc0
PoshCode ID: 1417
Published Date: 
Archived Date: 2009-10-26t15
---

# ad-promiscdetect - 

## Description

promiscuous mode check

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 <#================================================================================================
   NAME:                 	AD-PromiscDetect.ps1
   AUTHOR:			Marty J. Piccinich
   DATE CREATED: 		9/15/2009
   VERSION:  	          1.1
   HISTORY:         		1.0 9/15/2009 - Created
 			1.1 10/23/2009 - Added comment header
   PARAMETERS: None
   
   DESCRIPTION:
     Queries Active Directory (currently logged into) for all computer objects,
     pings the computer first, then queries WMI. The MSNdis_CurrentPackFilter class
     in the root\WMI namespace is used. Next, the NdisCurrentPacketFilter property to
     see if the NDIS_PACKET_TYPE_PROMISCUOUS bit (0x00000020) is set. This determines
     if the NIC is is promiscuous mode.
 
   REQUIREMENTS:
     Proper rights to query WMI\Root on the remote computers is necessary.
 
   NOTES:
     For more details, see the blog post:
     http://praetorianprefect.com/archives/2009/09/whos-being-promiscuous-in-your-active-directory/
     Credit to Harlan Carvey who provided information on NdisCurrentPacketFilter in his blog.
 	
   IMPORTANT:
 	You have a royalty-free right to use, modify, reproduce, and
 	distribute this script file in any way you find useful, provided that
 	you agree that the creator, owner above has no warranty, obligations,
 	or liability for such use.
 ================================================================================================#> 
 $ErrorActionPreference = "SilentlyContinue" 
 
 $PingTest = New-Object System.Net.NetworkInformation.Ping
 $Filter = "(&(ObjectCategory=computer))"
 $Searcher = New-Object System.DirectoryServices.DirectorySearcher($Filter)
 ForEach ($comp in $Searcher.Findall()) {
 	$strComputer = $comp.properties.item("Name")
 	write-host "Checking: $strComputer"
 	if ($PingTest.Send($strComputer).Status -eq "Success") {			
 		$colComputer = get-wmiObject -class "MSNdis_CurrentPacketFilter" -namespace "root\WMI" -comp $strComputer
 		if ($colComputer -eq $null) {
 			write-host "Couldn't connect to WMI" }
 		else {
 			foreach ($comp in $colcomputer) {
 				$val = $comp.NdisCurrentPacketFilter
 				if ($val -band 0x00000020) {
 					$inst = $comp.InstanceName
 					write-host "Interface: $inst"
 					write-host "The packetfilter value is above range" -foregroundcolor red -backgroundcolor yellow
 				}
 			}
 		} 
 	}
 	else { write-host "Could not ping, machine not queried." }
 }
`

