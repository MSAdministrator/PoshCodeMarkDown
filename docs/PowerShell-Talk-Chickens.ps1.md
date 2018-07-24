---
Author: rlewis
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 1406
Published Date: 
Archived Date: 2009-10-26t15
---

# powershell talk chickens - 

## Description

the powershell talk – chicken counter example.

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 #
 
 $credentials = Get-VICredentialStoreItem -File "c:\scripts\really_secure_file.xml"
 
 $datacenters = @{
     Name = "Datacenters"
     Expression = { Get-Datacenter | Measure-Object}
 }
 $hosts = @{
     Name = "Hosts"
     Expression = { Get-VMHost | Measure-Object }
 }
 $vms = @{
     Name = "VMs"
     Expression = { Get-VM | Measure-Object }
 }
 $clusters = @{
     Name = "Clusters"
     Expression = { Get-Cluster | Measure-Object }
 }
 
 ForEach ($creds in $credentials) {
 	
 	connect-viserver -Server $creds.Host -User $creds.User -Password $creds.Password
 	$output = $datacenter | Select-Object $datacenters, $hosts, $vms, $clusters
 $output = $datacenters | Select-Object $datacenters, $hosts, $vms, $clusters
  
 	disconnect-viserver -confirm:$false
     
 	$Body += "`nvCenter: " + $creds.Host
 	$Body += "`nDatacenters: " + $output.Datacenterss.count
 $Body += "`nDatacenters: " + $output.Datacenters.count
 
     $Body += "`nHosts: " + $output.Hosts.Count
     $Body += "`nVMs: " + $output.VM.Count
 $Body += "`nVMs: " + $output.VMS.Count
  
     $Body += "`nClusters: " + $output.Clusters.Count
     
 	$totalHosts += $output.Hosts.count
 	$totalVM += $output.VM.count
 $totalVM += $output.VMS.count
 	$totalDatacenters += $output.Datacenters.count
 	$totalClusters += $output.Clusters.count
 }
 
 $SmtpClient = new-object Net.Mail.SmtpClient("localhost")
 $Title = "Direct from your local VC: Weekly Statistics"
 
 $Body += "`n`nTotals:`n"
 $Body += "`nDatacenters: " + $totalDatacenters
 $Body += "`nHosts: " + $totalHosts
 $Body += "`nVMs: " + $totalVM
 $Body += "`nClusters: " + $totalClusters
 
 $SmtpClient.Send($From, $To, $Title, $Body)
`

