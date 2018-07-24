---
Author: hugopeeters
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 545
Published Date: 2008-08-22t09
Archived Date: 2014-11-07t16
---

# get-nicsettings - 

## Description

grab remote nic settings, including speed and duplex.

## Comments



## Usage



## TODO



## class

``

## Code

`#
 #
 #########################################################
 $serverName = Read-Host "Enter server name"
 $NicConfig = Get-WmiObject -Class Win32_NetworkAdapterConfiguration -ComputerName $serverName
 $myCol = @()
 ForEach ($Nic in $NicConfig)
 {
 If ($Nic.IPAddress -ne $null)
 {
 $myObj = "" | Select-Object Description, DHCPEnabled, IPAddress, IPSubnet, DefaultIPGateway, DNSServers, WINSServers, NICModel, SpeedDuplex
 $myObj.Description = $Nic.Description
 $myObj.DHCPEnabled = $Nic.DHCPEnabled
 $myObj.IPAddress = $Nic.IPAddress
 $myObj.IPSubnet = $Nic.IPSubnet
 $myObj.DefaultIPGateway = $Nic.DefaultIPGateway
 $myObj.DNSServers = $Nic.DNSServerSearchOrder
 $myObj.WINSServers = $Nic.WINSPrimaryServer,$Nic.WINSSecondaryServer
 $registry = [Microsoft.Win32.RegistryKey]::OpenRemoteBaseKey([Microsoft.Win32.RegistryHive]::LocalMachine, $serverName)
 $baseKey = $registry.OpenSubKey("SYSTEM\CurrentControlSet\Control\Class\{4D36E972-E325-11CE-BFC1-08002BE10318}")
 $subKeyNames = $baseKey.GetSubKeyNames()
 ForEach ($subKeyName in $subKeyNames)
 {
 $subKey = $baseKey.OpenSubKey("$subKeyName")
 $ID = $subKey.GetValue("NetCfgInstanceId")
 If ($ID -eq $Nic.SettingId)
 {
 $componentID = $subKey.GetValue("ComponentID")
 If ($componentID -match "ven_14e4")
 {
 $myObj.NICModel = "Broadcom"
 $requestedMediaType = $subKey.GetValue("RequestedMediaType")
 $enum = $subKey.OpenSubKey("Ndi\Params\RequestedMediaType\Enum")
 $myObj.SpeedDuplex = $enum.GetValue("$requestedMediaType")
 }
 ElseIf ($componentID -match "ven_8086")
 {
 $myObj.NICModel = "Intel"
 $SD = $subKey.GetValue("SpeedDuplex")
 $enum = $subKey.OpenSubKey("Ndi\Params\SpeedDuplex\Enum")
 $myObj.SpeedDuplex = $enum.GetValue("$SD")
 }
 ElseIf ($componentID -match "b06bdrv")
 {
 $myObj.NICModel = "HP"
 $SD = $subKey.GetValue("req_medium")
 $enum = $subKey.OpenSubKey("Ndi\Params\req_medium\Enum")
 $myObj.SpeedDuplex = $enum.GetValue("$SD")
 }
 Else
 {
 $myObj.NICModel = "unknown"
 $myObj.SpeedDuplex = "unknown"
 }
 }
 }
 $myCol += $myObj
 }
 }
 $myCol
`

