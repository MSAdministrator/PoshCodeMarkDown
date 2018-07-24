---
Author: juan carlos
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 3183
Published Date: 2012-01-24t05
Archived Date: 2012-01-27t05
---

# test-server - 

## Description

a test that checks for hostname, ip, domain, ping, wsman (with credssp if you wish) remote reg rpc and rdp

## Comments



## Usage



## TODO



## function

`test-server`

## Code

`#
 #
 Function Test-Server{
 [cmdletBinding()]
 param(
 	[parameter(Mandatory=$true,ValueFromPipeline=$true)]
 	[string[]]$ComputerName,
 	[parameter(Mandatory=$false)]
 	[switch]$CredSSP,
 	[Management.Automation.PSCredential] $Credential)
 	
 begin{
 	$total = Get-Date
 	$results = @()
 	if($credssp){if(!($credential)){Write-Host "must supply Credentials with CredSSP test";break}}
 }
 process{
     foreach($name in $computername)
     {
 	$dt = $cdt= Get-Date
 	Write-verbose "Testing: $Name"
 	$failed = 0
 	try{
 	$DNSEntity = [Net.Dns]::GetHostEntry($name)
 	$domain = ($DNSEntity.hostname).replace("$name.","")
 	$ips = $DNSEntity.AddressList | %{$_.IPAddressToString}
 	}
 	catch
 	{
 		$rst = "" |  select Name,IP,Domain,Ping,WSMAN,CredSSP,RemoteReg,RPC,RDP
 		$rst.name = $name
 		$results += $rst
 		$failed = 1
 	}
 	Write-verbose "DNS:  $((New-TimeSpan $dt ($dt = get-date)).totalseconds)"
 	if($failed -eq 0){
 	foreach($ip in $ips)
 	{
 	    
 		$rst = "" |  select Name,IP,Domain,Ping,WSMAN,CredSSP,RemoteReg,RPC,RDP
 	    $rst.name = $name
 		$rst.ip = $ip
 		$rst.domain = $domain
 		try{
             $socket = New-Object Net.Sockets.TcpClient($name, 3389)
 		  if($socket -eq $null)
 		  {
 			 $rst.RDP = $false
 		  }
 		  else
 		  {
 			 $rst.RDP = $true
 			 $socket.close()
 		  }
        }
        catch
        {
             $rst.RDP = $false
        }
 		Write-verbose "RDP:  $((New-TimeSpan $dt ($dt = get-date)).totalseconds)"
 	    if(test-connection $ip -count 1 -Quiet)
 	    {
 	        Write-verbose "PING:  $((New-TimeSpan $dt ($dt = get-date)).totalseconds)"
 			$rst.ping = $true
 				Test-WSMan $ip | Out-Null
 				$rst.WSMAN = $true
 				}
 			catch
 				{$rst.WSMAN = $false}
 				Write-verbose "WSMAN:  $((New-TimeSpan $dt ($dt = get-date)).totalseconds)"
 			{
 				try{
 					Test-WSMan $ip -Authentication Credssp -Credential $cred
 					$rst.CredSSP = $true
 					}
 				catch
 					{$rst.CredSSP = $false}
 				Write-verbose "CredSSP:  $((New-TimeSpan $dt ($dt = get-date)).totalseconds)"
 			}
 			{
 				[Microsoft.Win32.RegistryKey]::OpenRemoteBaseKey([Microsoft.Win32.RegistryHive]::LocalMachine, $ip) | Out-Null
 				$rst.remotereg = $true
 			}
 			catch
 				{$rst.remotereg = $false}
 			Write-verbose "remote reg:  $((New-TimeSpan $dt ($dt = get-date)).totalseconds)"
 			{	
 				$w = [wmi] ''
 				$w.psbase.options.timeout = 15000000
 				$w.path = "\\$Name\root\cimv2:Win32_ComputerSystem.Name='$Name'"
 				$w | select none | Out-Null
 				$rst.RPC = $true
 			}
 			catch
 				{$rst.rpc = $false}
 			Write-verbose "WMI:  $((New-TimeSpan $dt ($dt = get-date)).totalseconds)" 
 	    }
 		else
 		{
 			$rst.ping = $false
 			$rst.wsman = $false
 			$rst.credssp = $false
 			$rst.remotereg = $false
 			$rst.rpc = $false
 		}
 		$results += $rst	
 	}}
 	Write-Verbose "Time for $($Name): $((New-TimeSpan $cdt ($dt)).totalseconds)"
 	Write-Verbose "----------------------------"
 }
 }
 end{
 	Write-Verbose "Time for all: $((New-TimeSpan $total ($dt)).totalseconds)"
 	Write-Verbose "----------------------------"
 return $results
 }
 }
`

