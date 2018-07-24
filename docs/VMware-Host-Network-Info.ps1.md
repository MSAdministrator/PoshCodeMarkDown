---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 855
Published Date: 
Archived Date: 2009-02-10t03
---

# vmware host network info - 

## Description

the following script will add some nice host network information into an object which is exported to a csv file for passing to the network guys or can be used to find your server in that mess of cables that are always meaning to be tidied in the data center.

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 Connect-VIServer MYVISERVER
 $filename = "c:\DetailedNetworkInfo.csv"
 Write "Gathering VMHost objects"
 $vmhosts = Get-VMHost | Sort Name | Where-Object {$_.State -eq "Connected"} | Get-View
 $MyCol = @()
 foreach ($vmhost in $vmhosts){
  $ESXHost = $vmhost.Name
  Write "Collating information for $ESXHost"
  $networkSystem = Get-view $vmhost.ConfigManager.NetworkSystem
  foreach($pnic in $networkSystem.NetworkConfig.Pnic){
      $pnicInfo = $networkSystem.QueryNetworkHint($pnic.Device)
      foreach($Hint in $pnicInfo){
          $NetworkInfo = "" | select-Object Host, PNic, Speed, MAC, DeviceID, PortID, Observed, VLAN
          $NetworkInfo.Host = $vmhost.Name
          $NetworkInfo.PNic = $Hint.Device
          $NetworkInfo.DeviceID = $Hint.connectedSwitchPort.DevId
          $NetworkInfo.PortID = $Hint.connectedSwitchPort.PortId
          $record = 0
          Do{
              If ($Hint.Device -eq $vmhost.Config.Network.Pnic[$record].Device){
                  $NetworkInfo.Speed = $vmhost.Config.Network.Pnic[$record].LinkSpeed.SpeedMb
                  $NetworkInfo.MAC = $vmhost.Config.Network.Pnic[$record].Mac
              }
              $record ++
          }
          Until ($record -eq ($vmhost.Config.Network.Pnic.Length))
          foreach ($obs in $Hint.Subnet){
              $NetworkInfo.Observed += $obs.IpSubnet + " "
              Foreach ($VLAN in $obs.VlanId){
                  If ($VLAN -eq $null){
                  }
                  Else{
                      $strVLAN = $VLAN.ToString()
                      $NetworkInfo.VLAN += $strVLAN + " "
                  }
              }
          }
          $MyCol += $NetworkInfo
      }
  }
 }
 $Mycol | Sort Host, PNic | Export-Csv $filename -NoTypeInformation
`

