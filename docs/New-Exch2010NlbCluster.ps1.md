---
Author: jan egil ring
Publisher: 
Copyright: 
Email: jer@powershell.no
Version: 1.0
Encoding: ascii
License: cc0
PoshCode ID: 2333
Published Date: 
Archived Date: 2010-11-02t18
---

# new-exch2010nlbcluster - new-exch2010nlbcluster.ps1

## Description

script to create a nlb-cluster for the cas/hub-roles in exchange server 2010.

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 ###########################################################################
 #
 #
 #
 #
 #
 #
 ###########################################################################
 
 Import-Module NetworkLoadBalancingClusters
 
 $ClusterFqdn = Read-Host "Enter FQDN for the new cluster"
 $InterfaceName = Read-Host "Enter interface name for NLB-adapter"
 $ClusterPrimaryIP = Read-Host "Enter cluster primary IP"
 $ClusterPrimaryIPSubnetMask = Read-Host "Enter subnetmask for cluster primary IP"
 
 Write-Host "Choose cluster operation mode"
 Write-Host "1 - Unicast"
 Write-Host "2 - Multicast"
 Write-Host "3 - IGMP Multicast"
 switch (Read-Host "Enter the number for your chosen operation mode")
 {
 1 {$OperationMode = "unicast"}
 2 {$OperationMode = "multicastcast"}
 3 {$OperationMode = "igmpmulticast"}
 default {Write-Warning "Invalid option, choose '1', '2' or '3'";return}
 }
 
 $MSExchangeRPCPort = Read-Host "Enter static port configured for Microsoft Exchange RPC (MAPI)"
 $MSExchangeABPort = Read-Host "Enter static port configured for Microsoft Exchange Address book service"
 
 Write-Host "Creating NLB Cluster..." -ForegroundColor yellow
 New-NlbCluster -ClusterName $ClusterFqdn -InterfaceName $InterfaceName -ClusterPrimaryIP $ClusterPrimaryIP -SubnetMask $ClusterPrimaryIPSubnetMask -OperationMode $OperationMode
 
 Write-Host "Removing default port rule..." -ForegroundColor yellow
 Get-NlbClusterPortRule -HostName . | Remove-NlbClusterPortRule -Force
 
 Write-Host "Adding port rules for Exchange Server 2010 CAS" -ForegroundColor yellow
 Add-NlbClusterPortRule -Protocol Tcp -Mode Multiple -Affinity Single -StartPort 25 -EndPort 25 -InterfaceName $InterfaceName | Out-Null
 Write-Host "Added port rule for SMTP (tcp 25)" -ForegroundColor yellow
 Add-NlbClusterPortRule -Protocol Tcp -Mode Multiple -Affinity Single -StartPort 80 -EndPort 80 -InterfaceName $InterfaceName | Out-Null
 Write-Host "Added port rule for http (tcp 80)" -ForegroundColor yellow
 Add-NlbClusterPortRule -Protocol Tcp -Mode Multiple -Affinity Single -StartPort 110 -EndPort 110 -InterfaceName $InterfaceName | Out-Null
 Write-Host "Added port rule for POP3 (tcp 110)" -ForegroundColor yellow
 Add-NlbClusterPortRule -Protocol Tcp -Mode Multiple -Affinity Single -StartPort 135 -EndPort 135 -InterfaceName $InterfaceName | Out-Null
 Write-Host "Added port rule for RPC (tcp 135)" -ForegroundColor yellow
 Add-NlbClusterPortRule -Protocol Tcp -Mode Multiple -Affinity Single -StartPort 143 -EndPort 143 -InterfaceName $InterfaceName | Out-Null
 Write-Host "Added port rule for IMAP4 (tcp 143)" -ForegroundColor yellow
 Add-NlbClusterPortRule -Protocol Tcp -Mode Multiple -Affinity Single -StartPort 443 -EndPort 443 -InterfaceName $InterfaceName | Out-Null
 Write-Host "Added port rule for https (tcp 443)" -ForegroundColor yellow
 Add-NlbClusterPortRule -Protocol Tcp -Mode Multiple -Affinity Single -StartPort $MSExchangeRPCPort -EndPort $MSExchangeRPCPort -InterfaceName $InterfaceName | Out-Null
 Write-Host "Added port rule for MSExchange RPC (tcp $MSExchangeRPCPort)" -ForegroundColor yellow
 Add-NlbClusterPortRule -Protocol Tcp -Mode Multiple -Affinity Single -StartPort $MSExchangeABPort -EndPort $MSExchangeABPort -InterfaceName $InterfaceName | Out-Null
 Write-Host "Added port rule for MSExchange Address Book service (tcp $MSExchangeABPort)" -ForegroundColor yellow
 
 Write-Warning "Before adding additional nodes, make sure that NLB are installed and the NLB-adapter are configured with a static IP-address on the remote node"
 $additionalnodes = Read-Host "Add additional nodes to the cluster? Y/N"
 if ($additionalnodes -like "y"){
 do {
 $NodeFqdn = Read-Host "Enter FQDN for the additional node"
 $NewNodeInterface = Read-Host "Enter interface name for NLB-adapter on the additional node"
 Write-Host "Adding cluster node $NodeFqdn" -ForegroundColor yellow
 Get-NlbCluster -HostName . | Add-NlbClusterNode -NewNodeName $NodeFqdn -NewNodeInterface $NewNodeInterface
 $additionalnodes = Read-Host "Add additional nodes to the cluster? Y/N"
 } until ($additionalnodes -like "n") 
 }
`

