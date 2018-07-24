---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 4376
Published Date: 2015-08-08t17
Archived Date: 2015-08-11t00
---

# send-wakeonlan - 

## Description

send a wakeonlan packet to computers

## Comments



## Usage



## TODO



## script

`send-wakeonlan`

## Code

`#
 #
 $Computers = DATA {
   ConvertFrom-StringData -stringdata @'
     Server1 = 0E:A7:DE:AD:BE:EF
     Server2 = DE:AD:BE:EF:70:E5
 '@
 }
 
 Import-LocalizedData -BindingVariable Computers -ErrorAction SilentlyContinue -ErrorVariable NotLocalized
 if($NotLocalized) {
 	Write-Warning "No Server list found. Using hard-coded default servers:"
 	Write-Warning $NotLocalized[0]
 }
 
 function Send-WakeOnLan
 {
 	#.Synopsis
 	[CmdletBinding(SupportsShouldProcess=$true, ConfirmImpact="High")]
 	param(
 		[Parameter(Mandatory=$true,ParameterSetName="MacAddress",ValueFromPipeline=$true,ValueFromPipelineByPropertyName=$true,ValueFromRemainingArguments=$true)]
 		[string[]]$Computer
 	)
 
 
 	$MacAddresses = foreach($c in $Computer) {
 		if($output = foreach($pc in $Computers.GetEnumerator()) {
 			if($pc.Key -like $c -or $pc.Value -eq $c) { 
 				$pc.Value | Add-Member NoteProperty Name $pc.Name -Passthru
 			}
 		}) { $output } else { $c }
 	}
 
 
 	foreach($mac in $MacAddresses) {
 		$MacAddress = [regex]::Match($mac, '^(?:([0-9A-F]{2}).?){6}$')
 		if(!$MacAddress.Success) {
 			Write-Warning "$($mac.Name) $mac does not have a valid MacAddress. WakeOnLan skipped."
 			continue
 		}
 
 	   if($PSCmdlet.ShouldProcess( "Waking the computer $mac $($mac.Name)",
 	                               "Wake the computer $mac $($mac.Name)?",
 	                               "Waking Computers" )) {
 
 			$MacAddress = $MacAddress.Groups[1].Captures.Value | % { [byte]"0x$_" }
 
 			$UDPclient = new-Object System.Net.Sockets.UdpClient
 			$UDPclient.Connect(([System.Net.IPAddress]::Broadcast),4000)
 			$packet = [byte[]](,0xFF * 102)
 			6..101 |% { $packet[$_] = $mac[($_%6)]}
 			$null = $UDPclient.Send($packet, $packet.Length)
 		}
 	}
 }
`

