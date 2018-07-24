---
Author: leon scheltema
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5705
Published Date: 2015-01-23t12
Archived Date: 2015-01-25t23
---

# step04-copy_vlan_clust - 

## Description

script to copy standard vswitch config across all host within the cluster

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 
 $sourceVI = "New vCenter"
 $BASEHost = "Base host"
 
 Connect-VIServer "$sourceVI"
 
 $NEWHost = Get-Cluster "Base host Cluster" | Get-VMhost
 
 Foreach ($hosts in $NEWHost) {
 Get-VMHost -Name $BASEHost |Get-VirtualSwitch |Foreach {
    If ((Get-VMHost -Name $hosts |Get-VirtualSwitch -Name $_.Name-ErrorAction SilentlyContinue)-eq $null){
        Write-Host "Creating Virtual Switch $($_.Name)"
        $NewSwitch = Get-VMHost -Name $hosts |New-VirtualSwitch -Name $_.Name-NumPorts $_.NumPorts-Mtu $_.Mtu
        $vSwitch = $_
                     }
                    $_ |Get-VirtualPortGroup |Foreach {
                        If ((Get-VMHost -Name $hosts |Get-VirtualPortGroup -Name $_.Name-ErrorAction SilentlyContinue)-eq $null){
                            Write-Host "Creating Portgroup $($_.Name)"
                            $NewPortGroup = Get-VMHost -Name $hosts |Get-VirtualSwitch -Name $vSwitch |New-VirtualPortGroup -Name $_.Name-VLanId $_.VLanID
                         }
                     }
 }
 }
 
 Disconnect-VIServer -server "$sourceVI" -Force -Confirm:$false
`

