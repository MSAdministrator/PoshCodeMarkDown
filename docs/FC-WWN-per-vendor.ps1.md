---
Author: leon scheltema
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 4893
Published Date: 2015-02-12t10
Archived Date: 2015-05-22t02
---

# fc wwn per vendor - 

## Description

get’s fc adapter wwn’s listed per vendor of esxi hosts per cluster

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 
 $VC1 = ""
 $cluster = ""
 
 Connect-VIServer "$VC1"
 
 foreach ($esx in $scope){
 Write-Host "Host:", $esx
 $hbas = Get-VMHostHba -VMHost $esx -Type FibreChannel
 foreach ($hba in $hbas){
 $wwpn = "{0:x}" -f $hba.PortWorldWideName
 Write-Host `t $hba.Device, "|", $hba.model, "|", "World Wide Port Name:" $wwpn
 }}
 
 Disconnect-VIServer -server "$VC1" -Force -Confirm:$false
`

