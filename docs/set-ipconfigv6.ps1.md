---
Author: chriskenis
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 3925
Published Date: 2013-01-31t14
Archived Date: 2013-02-04t23
---

# set-ipconfigv6 - 

## Description

ye old remote ip config script but now with pipeline processing

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 param(
 [Parameter(Position=0,ValueFromPipeline=$True,ValueFromPipelineByPropertyName=$true)]
 [alias("Name","ComputerName")][string[]]$Computers = @($env:computername),
 $Domain = "domain.local",
 $DNSSuffix = @("domain.local,sub.domain.local,domain.com"),
 $DNSServers = @("10.10.0.1", "10.10.0.2", "10.10.0.3", "10.10.0.4"),
 $WINSServers = @("10.10.0.5", "10.10.0.6"),
 $Gateway = @("10.10.255.254"),
 [switch] $ChangeSettings,
 [switch] $EnableDHCP,
 [switch] $IpRelease, 
 [switch] $BatchReport
 )
 
 process{
 foreach ($Computer in $Computers){
 	If (Test-connection $Computer -quiet -count 1 -EA stop){
 		Try {
 			[array]$NICs = (Get-WMIObject -Class Win32_NetworkAdapterConfiguration -Computername $Computer -Filter "IPEnabled = TRUE" -EA Stop)
 			}
 		Catch {
 			Write-Warning "$($error[0])"
 			Write-Output "$("INACCESSIBLE: ")$($nl)$($Computer)"
 			Write-Host $nl
 			continue
 			}
 		$NICindex = $NICs.count
 		Write-Host "$nl Selection for $($Computer) : $nl"
 		For ($i=0;$i -lt $NICindex; $i++) {
 			Write-Host -ForegroundColor Green "$i --> $($NICs[$i].Description)"
 			Write-Output $(ShowDetails $NICs[$i] $Computer)
 			}
 		$nl
 		if ($BatchReport){continue}
 		Write-Host -ForegroundColor Green "q --> Quit" $nl
 		Do {
 			$SelectIndex = Read-Host "Select connection by number or 'q' (=default) to quit"
 			Switch -regex ($SelectIndex){
 				"^q.*" 	{$SelectIndex="quit"; $kip = $true}
 				"\d" 	{$SelectIndex = $SelectIndex -as [int];$kip = $false}
 				"^\s*$" {$SelectIndex="quit"; $kip = $true}
 			}
 		}
 		Until (($SelectIndex -lt $NICindex) -OR $SelectIndex -like "q*")
 		$nl
 		Write-Host "You selected: $SelectIndex" $nl
 		If ($kip) {continue}
 		Else {ProcessNIC $NICs[$SelectIndex] $Computer}
 	}
 	else {
 		Write-warning "$Computer cannot be reached"
 		}
 	}
 }
 
 begin{
 $nl = [Environment]::NewLine
 
 Function ProcessNIC($NIC, $Computer){
 	If ($ChangeSettings){
 		If (-NOT ([Security.Principal.WindowsPrincipal] [Security.Principal.WindowsIdentity]::GetCurrent()).IsInRole([Security.Principal.WindowsBuiltInRole] "Administrator")){
 			Write-Warning "You need Administrator rights to run this script!"
 			Break
 		}
 		If ($IpRelease){
 			#$NIC.ReleaseDHCPLease
 			$NIC.RenewDHCPLease
 			}
 		Else{
 			ChangeIPConfig $NIC $Computer
 			}
 			start-sleep -s 2
 			Write-Host $nl "    ====NEW SETTINGS====" $nl
 			$UpdatedNIC = Get-WMIObject -Class Win32_NetworkAdapterConfiguration -Computername $Computer -Filter "Index=$($NIC.Index)"
 			Write-Output $(ShowDetails $UpdatedNIC $Computer)$($nl)
 		}
 	Else{
 			$nl
 			Write-Warning "For changing settings add -ChangeSettings as parameter, if not this script is output only"
 			$nl
 		}
 }
 
 Function ChangeIPConfig($NIC, $Computer){
 	if ($EnableDHCP){
 		$NIC.EnableDHCP()
 		$NIC.SetDNSServerSearchOrder()
 		}
 	else{
 		$DNSServers = Get-random $DNSservers -Count $DNSServers.Length
 		$NIC.SetDNSServerSearchOrder($DNSServers) | Out-Null
 		#$x = 0
 		#$IPaddress = @()
 		#$NetMask = @()
 		#$Gateway = @()
 		#$Metric = @()
 			#$IPaddress[$x] = $NIC.IPAddress[$x]
 			#$NetMask[$x] = $NIC.IPSubnet[$x]
 			#$Gateway[$x] = $NIC.DefaultIPGateway[$x]
 			#$Metric[$x] = $NIC.GatewayCostMetric[$x]
 			#$x++
 		#}
 		#$NIC.EnableStatic($IPaddress, $NetMask)
 		#$NIC.SetGateways($Gateway, $Metric)
 		#$NIC.SetWINSServer($WINSServers)
 		}
 	$NIC.SetDynamicDNSRegistration("TRUE") | Out-Null
 	$NIC.SetDNSDomain("") | Out-Null
 	$registry = [WMIClass]"\\$computer\root\default:StdRegProv"
 	$HKLM = [UInt32] "0x80000002"
 	$registry.SetStringValue($HKLM, "SYSTEM\CurrentControlSet\Services\TCPIP\Parameters", "SearchList", $DNSSuffix) | Out-Null
 }
 
 Function ShowDetails($NIC, $Computer){
 	Write-Output "$($nl)$(" IP settings on: ")$($Computer)$($nl)$($nl)$(" for") $($NIC.Description)$(":")$($nl)"
 	Write-Output "$("Hostname = ")$($NIC.DNSHostName)"
 	Write-Output "$("DNSDomain= ")$($NIC.DNSDomain)"
 	Write-Output "$("Domain DNS Registration Enabled = ")$($NIC.DomainDNSRegistrationEnabled)"
 	Write-Output "$("Full DNS Registration Enabled = ")$($NIC.FullDNSRegistrationEnabled)"
 	Write-Output "$("DNS Domain Suffix Search Order = ")$($NIC.DNSDomainSuffixSearchOrder)"
 	Write-Output "$("MAC address = ")$($NIC.MACAddress)"
 	Write-Output "$("DHCP enabled = ")$($NIC.DHCPEnabled)"
 	$x = 0
 	foreach ($IP in $NIC.IPAddress){
 		Write-Output "$("IP address $x =")$($NIC.IPAddress[$x])$("/")$($NIC.IPSubnet[$x])"
 		$x++
 	}
 	Write-Output "$("Default IP Gateway = ")$($NIC.DefaultIPGateway)"
 	Write-Output "$("DNS Server Search Order = ")$($NIC.DNSServerSearchOrder)"
 }
 }
`

