---
Author: chriskenis
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 3404
Published Date: 2012-05-08t01
Archived Date: 2012-08-09t14
---

# set-ipconfigv2 - 

## Description

second version of set ip config thru wmi,

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 param(
 [string[]] $Computers = $env:computername,
 [switch] $ChangeSettings,
 [switch] $EnableDHCP,
 [switch] $Batch
 )
 $nl = [Environment]::NewLine
 if ($ChangeSettings -or $EnableDHCP){
 	If (-NOT ([Security.Principal.WindowsPrincipal] [Security.Principal.WindowsIdentity]::GetCurrent()).IsInRole([Security.Principal.WindowsBuiltInRole] "Administrator")){
 		Write-Warning "You need Administrator rights to run this script!"
 		Break
 	}
 }
 else{
 $nl
 Write-Warning "For changing settings add -ChangeSettings as parameter, if not this script is output only"
 }
 
 $Domain = "domain.local"
 $DNSSuffix = @("domain.local", "domain.com")
 $DNSServers = @("10.10.0.1", "10.10.0.2", "10.10.0.3", "10.10.0.4")
 $WINSServers = @("10.10.0.5", "10.10.0.6")
 $Gateway = @("10.10.255.254")
 Function NewNICDetails($NIC, $Computer){
 	$UpdatedNIC = Get-WMIObject Win32_NetworkAdapterConfiguration -Computername $Computer | where{$_.Index -eq $NIC.Index}
 	ShowDetails $UpdatedNIC
 }
 
 Function ChangeIPConfig($NIC){
 	if ($EnableDHCP){$NIC.EnableDHCP()}
 	#$NIC.SetGateways($Gateway)
 	#$NIC.SetWINSServer($WINSServers)
 	$DNSServers = Get-random $DNSservers -Count 4
 	$NIC.SetDNSServerSearchOrder($DNSServers)
 	$NIC.SetDynamicDNSRegistration("TRUE")
 	$NIC.SetDNSDomain($Domain)
 	$registry = [WMIClass]"\\$computer\root\default:StdRegProv"
 	$HKLM = [UInt32] "0x80000002"
 	$registry.SetStringValue($HKLM, "SYSTEM\CurrentControlSet\Services\TCPIP\Parameters", "SearchList", $DNSSuffix)
 }
 
 Function ShowDetails($NIC){
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
 foreach ($Computer in $Computers){
 	if (Test-connection $Computer -quiet -count 1){
 	Try {
 		[array]$NICs = Get-WMIObject Win32_NetworkAdapterConfiguration -Computername $Computer -EA Stop | where{$_.IPEnabled -eq "TRUE"}
 		}
 	Catch {
 		Write-Host $nl "    ====INACCESIBLE====" $nl
 		Write-Warning "$($error[0])"
 		Write-Output "$($nl)$($Computer.ToUpper())$(" is INACCESIBLE")"
 		Write-Host $nl
 		continue
 		}
 	$NICindex = $NICs.count
 	$SelectIndex = 0
 	if ($NICindex -gt 1){
 		if ($Batch){
 			Write-Host $nl "    ====SKIPPED====" $nl
 			Write-Output "$($nl)$($Computer.ToUpper())$(" skipped due to multiple NICs")"
 			continue
 			}
 		else{
 			Write-Host "$nl Selection for " $Computer.ToUpper() ": $nl"
 			For ($i=0;$i -lt $NICindex; $i++) {
 				Write-Host -ForegroundColor Green $i --> $($NICs[$i].Description)
 				}
 				Write-Host -ForegroundColor Green q --> Quit
 				Write-Host $nl
 			Do {
 				$SelectIndex = Read-Host "Select connection (default = $SelectIndex) or 'q' to quit"
 				If ($SelectIndex -NotLike "q*"){$SelectIndex = $SelectIndex -as [int]}
 			}
 			Until (($SelectIndex -lt $NICindex -AND $SelectIndex -match "\d") -OR $SelectIndex -Like "q*")
 			If ($SelectIndex -Like "q*"){continue}
 			}
 		}
 	Write-Host $nl "     ====BEFORE====" $nl
 	Write-Output "$($nl)$(" IP settings on:")$($Computer.ToUpper())$($nl)$($nl)$(" for ")$($NICs[$SelectIndex].Description)$(":")"
 	Write-Output $(ShowDetails $NICs[$SelectIndex])$($nl)
 	If ($ChangeSettings){
 		ChangeIPConfig $NICs[$SelectIndex]
 		Write-Host $nl "    ====AFTER====" $nl
 		Write-Output "$($nl)$(" IP settings on:")$($Computer.ToUpper())$($nl)$($nl)$(" for ") $($NICs[$SelectIndex].Description)$(":")"
 		Write-Output $(NewNICDetails $NICs[$SelectIndex] $Computer)$($nl)
 		}
 	}
 }
`

