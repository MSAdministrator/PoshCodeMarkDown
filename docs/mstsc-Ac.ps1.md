---
Author: justin dearing
Publisher: 
Copyright: 
Email: 
Version: 1.0
Encoding: ascii
License: mitl
PoshCode ID: 3154
Published Date: 2012-01-07t17
Archived Date: 2012-01-14t07
---

# mstsc-ac.ps1 - 

## Description

pings a host until it responds, tries to connect to the rdp port, and then when that succeeds, launches a remote desktop connection via mstsc.exe. i discuss this version here http

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 <#
 .SYNOPSIS
 mstsc-Ac.ps1 (Version 1.0, 7 Jan 2012)
 The author may be contacted via zippy1981@gmail.com
 The latest authoritative version of this script is always available at
 http://bit.ly/mstsc-Ac
 .DESCRIPTION 
 This script will see if a host is up and listening on a given port, and start a 
 remote desktop connection to it. The idea is you run this script after rebooting a windows server
 .EXAMPLE
 .\mstsc-Ac.ps1 192.168.0.2
 Starts an RDP connection on 192.168.0.2
 .EXAMPLE
 .\mstsc-Ac.ps1 192.168.0.2 3390
 Starts an RDP connection on 192.168.0.2 port 3390 .
 .EXAMPLE
 mstsc-Ac.ps1 192.168.0.2 -AdditionalParameters "/w:1000 /h:500"
 Starts an RDP connection on 192.168.0.2 with a width of 1000 pixels and a height of 500 pixels.
 .Notes
    Copyright (c) 2012 Justin Dearing
 
    Permission is hereby granted, free of charge, to any person obtaining a copy 
    of this software and associated documentation files (the "Software"), to deal
    in the Software without restriction, including without limitation the rights 
    to use, copy, modify, merge, publish, distribute, sublicense, and/or sell 
    copies of the Software, and to permit persons to whom the Software is 
    furnished to do so, subject to the following conditions:
 
    The above copyright notice and this permission notice shall be included in 
    all copies or substantial portions of the Software.
 
    THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR 
    IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, 
    FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE 
    AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER 
    LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
    OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE 
    SOFTWARE.
    *****************************************************************************
    NOTE: YOU MAY *ALSO* DISTRIBUTE THIS FILE UNDER ANY OF THE FOLLOWING...
    PERMISSIVE LICENSES:
    BSD:	 http://www.opensource.org/licenses/bsd-license.php
    MIT:   http://www.opensource.org/licenses/mit-license.html
    RECIPROCAL LICENSES:
    GPL 2: http://www.gnu.org/copyleft/gpl.html
    *****************************************************************************
    LASTLY: THIS IS NOT LICENSED UNDER GPL v3 (although the above are compatible)
 #>
 param(
 	[string] $HostOrIpAddress = "",
 	[int] $Port = 3389,
 	[string] $AdditionalParameters = "/f"
 )
 	[string] $userName = $(Read-Host -Prompt "Username"),
 	[string] $password = $(
 		[System.Runtime.InteropServices.Marshal]::SecureStringToBSTR(
 			(Read-Host -Prompt "Password" -AsSecureString)
 		)
 	)
 #>
 
 if ([String]::IsNullOrEmpty($HostOrIpAddress)) {
 	$lastHost = (Get-ItemProperty 'HKCU:\Software\Microsoft\Terminal Server Client\Default' 'MRU0').MRU0
 	
 	$hostInput = (Read-Host -Prompt "Host (Default $($lastHost))").Split(':')
 	if ([String]::IsNullOrEmpty($hostInput)) { $hostInput = $lastHost }
 	$HostOrIpAddress = $hostInput[0];
 	if ($host.Length -gt 1) { $Port = $hostInput[1] }
 }
 
 $successfulConsecutivePingCount = 5
 $ping = New-Object System.Net.NetworkInformation.Ping
 $rdpPortListening = $false;
 
 
 while (-not $rdpPortListening) {
 	for ($i = 0; $i -lt $successfulConsecutivePingCount; $i++) {
 		$result = $ping.Send($HostOrIpAddress, $pingTimeout);
 		if ($result.Status -eq 'Success') {
 			Write-Host "Reply from $($result.Address) Round Trip Time $($result.RoundTripTime)"
 		} else {
 			Write-Host "Request Timed out. Sleeping for $($sleepTime) seconds."
 			$i = 0;
 			Start-Sleep $sleepTime
 		}
 		#$ping.Send('74.125.115.107', 500);
 	}
 	try {
 		$socket = New-Object Net.Sockets.TcpClient($HostOrIpAddress, $Port)
 		if ($socket.Connected) { 
 			$rdpPortListening = $true
 			Write-Host "RDP Service appears to be up"
 		}
 		$socket.Close()
 	}
 	catch [System.Management.Automation.MethodInvocationException] {
 		if ($_.Exception.InnerException.GetType() -eq [System.Net.Sockets.SocketException]) {
 			Write-Host $_.Exception.InnerException.Message
 			Start-Sleep $sleepTime
 		}
 		else {
 			throw $_.Exception.InnerException
 		}
 		
 	}
 }
 
 Invoke-Expression "mstsc /v $($HostOrIpAddress):$($Port) $($AdditionalParameters)"
`

