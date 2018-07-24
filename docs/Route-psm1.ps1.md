---
Author: fdibot
Publisher: 
Copyright: 
Email: 
Version: 0.2
Encoding: utf-8
License: cc0
PoshCode ID: 3464
Published Date: 2012-06-20t08
Archived Date: 2012-06-22t06
---

# route.psm1 - 

## Description

this module provides 3 cmdlets

## Comments



## Usage



## TODO



## module

`start-execute`

## Code

`#
 #
 <#
 .SYNOPSIS
 This module provides Cmdlets that help to manage Network routes.
 .DESCRIPTION
 This module provides 3 Cmdlets : Get-Route, Add-Route and Remove-Route.
 This module use WMI Classes to gathering information, be sure your account used to launch the script have the right to query them.
 This module is based on the route.exe program to add and remove routes.
 This module requires WinRM setted on all devices, check http://msdn.microsoft.com/en-us/library/windows/desktop/aa384372(v=vs.85).aspx to enable and configure it.
 This program is located on C:\Windows\System32\Route.exe (Test on Windows Server 2008 SP2, 2008 R2, 7)
 .NOTES
 NAME: Routes.psm1
 AUTHOR: Fabien DIBOT (Twitter: @fdibot)
 LASTEDIT: 6/20/2012
 VERSION: 0.2
 .LINK
 http://posh.netau.net
 http://www.powershell-scripting.com
 #>
 Function Start-Execute {
 <#
 .SYNOPSIS
 This function create a process and release prompt after the execution finish
 This function is only used in Local
 .PARAMETER Setup
 This parameter must be a String, it's the path to the .exe of the program
 .PARAMETER param
 This parameter must be a String, it's switch or parameters used by the program
 #>
 param (
 [Parameter(Mandatory=$true)]
 [String]$setup,
 [Parameter(Mandatory=$true)]
 [String]$param
 )
 Try {
 $Process = [System.Diagnostics.Process]::Start($setup, $param)
 $Process.waitforexit()
 }
 Catch {
 Write-Error "Error Executing Route.exe"
 return $false
 }
 }
 Function Get-ErrorCode {
 <#
 .SYNOPSIS
 This function return the error labels with ID entry
 This function is only used in Local
 .PARAMETER ErrorID
 This parameter must be an integer, it the error number
 .SWITCH Availability
 This switch, design what the errorcode is for getting Availability labels
 .SWITCH ConfigMngr
 This switch, design what the errorcode is for getting ConfigMngr labels
 #>
 param (
 [Parameter(Mandatory=$true)]
 [Int]$ErrorID,
 [Parameter(Mandatory=$false)]
 [Switch]$Availability,
 [Parameter(Mandatory=$false)]
 [Switch]$ConfigMngr
 )
 if ($availability) {
 $ErrorCode = Switch ($ErrorID) {
 1 {"Other"; Break}
 2 {"Unknown"; Break}
 3 {"Running Or Full Power"; Break}
 4 {"Warning"; Break}
 5 {"In Test"; Break}
 6 {"Not Applicable"; Break}
 7 {"Power Off"; Break}
 8 {"Off Line"; Break}
 9 {"Off Duty"; Break}
 10 {"Degraded"; Break}
 11 {"Not Installed"; Break}
 12 {"Install Error"; Break}
 13 {"Power Save - Unknown"; Break}
 14 {"Power Save - Low Power Mode"; Break}
 15 {"Power Save - Standby"; Break}
 16 {"Power Cycle"; Break}
 17 {"Power Save - Warning"; Break}
 }
 }
 elseif ($ConfigMngr) {
 $ErrorCode = Switch ($ErrorID) {
 0 {"Device is working properly"; Break}
 1 {"Device is not configured correctly."; Break}
 2 {"Windows cannot load the driver for this device."; Break}
 3 {"Driver for this device might be corrupted, or the system may be low on memory or other resources."; Break}
 4 {"Device is not working properly. One of its drivers or the registry might be corrupted."; Break}
 5 {"Driver for the device requires a resource that Windows cannot manage."; Break}
 6 {"Boot configuration for the device conflicts with other devices."; Break}
 7 {"Cannot filter."; Break}
 8 {"Driver loader for the device is missing."; Break}
 9 {"Device is not working properly. The controlling firmware is incorrectly reporting the resources for the device."; Break}
 10 {"Device cannot start."; Break}
 11 {"Device failed."; Break}
 12 {"Device cannot find enough free resources to use."; Break}
 13 {"Windows cannot verify the device's resources."; Break}
 14 {"Device cannot work properly until the computer is restarted."; Break}
 15 {"Device is not working properly due to a possible re-enumeration problem."; Break}
 16 {"Windows cannot identify all of the resources that the device uses."; Break}
 17 {"Device is requesting an unknown resource type."; Break}
 18 {"Device drivers must be reinstalled."; Break}
 19 {"Failure using the VxD loader."; Break}
 20 {"Registry might be corrupted."; Break}
 21 {"System failure. If changing the device driver is ineffective, see the hardware documentation. Windows is removing the device."; Break}
 22 {"Device is disabled."; Break}
 23 {"System failure. If changing the device driver is ineffective, see the hardware documentation."; Break}
 24 {"Device is not present, not working properly, or does not have all of its drivers installed."; Break}
 25 {"Windows is still setting up the device."; Break}
 26 {"Windows is still setting up the device."; Break}
 27 {"Device does not have valid log configuration."; Break}
 28 {"Device drivers are not installed."; Break}
 29 {"Device is disabled. The device firmware did not provide the required resources."; Break}
 30 {"Device is using an IRQ resource that another device is using."; Break}
 31 {"Device is not working properly. Windows cannot load the required device drivers."; Break}
 default {"Device is working properly"}
 }
 }
 return $ErrorCode
 }
 Function Get-Route {
 <#
 .SYNOPSIS
 This function use WMI class to list all routes on the target computer
 .PARAMETER ComputerName
 This parameter must be a String, you can pass it to the pipe.
 .EXAMPLE
 Get-Route -ComputerName toto
 Get-Content "C:\Path\To\List\Servers.txt" | Get-Route
 Get-Route | ? { $_.InterfaID -eq 1 } | Export-Csv Path\To\test.csv
 #>
 param(
 [Parameter(Position=0,Mandatory=$false,ValueFromPipeline=$true)]
 [alias('cn','computer', 'ComputerName')]
 [String]$server = $env:computername
 )
 Try {
 Write-Debug "Gathering Route information from $Server..."
 Invoke-Command -ComputerName $server -Scriptblock {
 $InterfaceIndexArray = @()
 Get-WmiObject Win32_NetworkAdapterConfiguration| % {
 $InterfaceIndexArray += New-Object PSObject -property @{
 'index' = $_.InterfaceIndex;
 'Description' = $_.Description
 }
 }
 Get-WmiObject win32_IP4RouteTable | % {
 $route = $_
 New-Object PSObject -property @{'Name' = $route.Name;
 'Age' = $route.Age;
 'Caption' = $route.Caption;
 'Destination' = $route.Destination;
 'Mask' = $route.Mask;
 'Metric' = $route.Metric1;
 'Gateway' = $route.NextHop;
 'ProtocolID' = $route.Protocol;
 'Protocol' = Switch ($route.Protocol) {
 1 {"Other"; Break}
 2 {"Local"; Break}
 3 {"Netmgmt"; Break}
 4 {"icmp"; Break}
 5 {"egp"; Break}
 6 {"ggp"; Break}
 7 {"hello"; Break}
 8 {"rip"; Break}
 9 {"is-is"; Break}
 10 {"es-is"; Break}
 11 {"CiscoIgrp"; Break}
 12 {"bbnSpfIgp"; Break}
 13 {"ospf"; Break}
 14 {"bgp"; Break}
 default {"Other"}
 }
 'Status' = $route.Status;
 'Information' = $route.Information;
 'InterfaceID' = $$route.InterfaceIndex
 'Interface' = ($InterfaceIndexArray | ? { $_.index -eq $route.InterfaceIndex } ).Description ;
 'TypeID' = $route.Type
 'Type' = Switch ($route.Type) {
 1 {"Other"; Break}
 2 {"Invalid"; Break}
 3 {"Direct"; Break}
 4 {"Indirect"; Break}
 default {"Other"}
 }
 }
 }
 }
 }
 Catch {
 Write-Error "Error Executing the remote command. Error: $($_.Exception.Message)"
 }
 }
 Function Add-Route {
 <#
 .SYNOPSIS
 This function use route.exe to add Routes on target computer
 .PARAMETER ComputerName
 This parameter must be a String, you can pass it to the pipe.
 .PARAMETER Destination
 It's the network you want to reach
 .PARAMETER Mask
 It's the subnet Mask for reaching the network
 .PARAMETER Gateway
 It's the gateway to reach the network
 .PARAMETER Metric
 It's the metric of this route
 .PARAMETER Interface
 It's the network card which will receive the route. By default it's the first listed network card
 .PARAMETER Persistant
 It's a swith to significate if the route should be persistant or not.
 .EXAMPLE
 Add-Route -Server toto -destination 192.168.0.0 -mask 255.255.0.0 -gateway 192.168.0.254 -metric 105 -interface 1 -persistant
 Add-Route -destination 192.168.128.0 -mask 255.255.0.0 -gateway 192.168.0.254 -interface 12
 Get-Content "C:\Path\To\List\Servers.txt" | Add-Route -destination 192.168.128.0 -mask 255.255.0.0 -gateway 192.168.0.254 -interface 15
 #>
 param (
 [Parameter(Position=0,Mandatory=$false,ValueFromPipeline=$true)]
 [String]$server = $env:computername,
 [Parameter(Mandatory=$true)]
 [IpAddress]$Destination,
 [Parameter(Mandatory=$true)]
 [IpAddress]$Mask,
 [Parameter(Mandatory=$true)]
 [IpAddress]$Gateway,
 [Parameter(Mandatory=$false)]
 [Int]$metric,
 [Parameter(Mandatory=$true)]
 [Int]$Interface,
 [Parameter(Mandatory=$false)]
 [Switch]$Persistant
 )
 if ($Persistant) { $persit = "-p " }
 if ($metric) { $met = " METRIC " + $metric }
 if ($Interface) { $int = " IF " + $Interface }
 $param = "ADD " + $persist + $Destination + " Mask " + $Mask + " " + $Gateway + $met + $int
 $setup = "route.exe"
 Try {
 Write-Debug "Adding the route $Destination to interface n° $Interface"
 Invoke-Command -ComputerName $server -Scriptblock -ErroAction Silentlycontinue {
 if (Test-Path ("C:\WINDOWS\system32\route.exe")) {
 Write-Debug "Checking if network Adaptater $Interface is working properly..."
 $InterfaceIndexArray = @()
 Get-WmiObject Win32_NetworkAdapter | % {
 $InterfaceIndexArray += New-Object PSObject -property @{
 'index' = $_.InterfaceIndex;
 'Availability' = $_.Availability;
 'ConfigManagerErrorCode' = $_.ConfigManagerErrorCode;
 'Description' = $_.Description
 }
 }
 $CheckedInterface = $InterfaceIndexArray | ? { $_.index -eq $Interface }
 if ($CheckedInterface) {
 if ( ( $CheckedInterface.Availability -eq 3 ) -and ( $CheckedInterface.ConfigManagerErrorCode -eq 0 ) ) {
 Write-Debug "$($CheckedInterface.Description) status: OK. Adding the route"
 Start-Execute -setup $setup -param $param
 return $true
 }
 else {
 $ErrorAvailable = Get-ErrorCode -ErrorId $CheckedInterface.Availability -Availability
 $ErrorCode = Get-ErrorCode -ErrorId $CheckedInterface.ConfigManagerErrorCode -ConfigMngr
 Write-Error "$($CheckedInterface.Description) Availability: $ErrorAvailable - Error: $ErrorCode"
 return $false
 }
 }
 else {
 Write-Error "Network Interface $Interface doesn't exists."
 return $false
 }
 }
 else {
 Write-Error "Route.exe missing in C:\Windows\System32."
 return $false
 }
 }
 }
 Catch {
 Write-Error "Error Executing the remote command. Error: $($_.Exception.Message)"
 return $false
 }
 }
 Function Remove-Route {
 <#
 .SYNOPSIS
 This function use route.exe to remove Routes on target computer
 .PARAMETER ComputerName
 This parameter must be a String, you can pass it to the pipe.
 .PARAMETER Destination
 It's the network you want to reach. You can use * or ?
 .PARAMETER Mask
 It's the subnet Mask for reaching the network
 .PARAMETER Gateway
 It's the gateway to reach the network
 .PARAMETER Metric
 It's the metric of this route
 .PARAMETER Interface
 It's the network card which will receive the route. By default it's the first listed network card
 .EXAMPLE
 Remove-Route -Server toto -destination 192.168.0.0 -mask 255.255.0.0 -gateway 192.168.0.254 -metric 105 -interface 11
 Remove-Route -destination 192.168.128.0 -mask 255.255.0.0 -gateway 192.168.0.254 -interface 15
 Get-Content "C:\Path\To\List\Servers.txt" | Remove-Route -destination 192.168.128.0 -mask 255.255.0.0 -gateway 192.168.0.254 -interface 22
 #>
 param (
 [Parameter(Position=0,Mandatory=$false,ValueFromPipeline=$true)]
 [String]$server = $env:computername,
 [Parameter(Mandatory=$true)]
 [String]$Destination,
 [Parameter(Mandatory=$false)]
 [String]$Mask,
 [Parameter(Mandatory=$false)]
 [String]$Gateway,
 [Parameter(Mandatory=$false)]
 [Int]$metric,
 [Parameter(Mandatory=$true)]
 [Int]$Interface
 )
 if ($metric) { $met = " METRIC " + $metric }
 if ($Interface) { $int = " IF " + $Interface }
 if ($Mask) { $SubnetMask = " MASK " + $Mask }
 if ($Gateway) { $gate = " " + $gateway }
 $param = "DELETE " + $Destination + $SubnetMask + $Gate + $met + $int
 $setup = "route.exe"
 Try {
 Write-Debug "Removing the route $Destination to interface n° $Interface"
 Invoke-Command -ComputerName $server -Scriptblock -ErroAction Silentlycontinue {
 if (Test-Path ("C:\WINDOWS\system32\route.exe")) {
 Write-Debug "Checking if network Adaptater $Interface is working properly..."
 $InterfaceIndexArray = @()
 Get-WmiObject Win32_NetworkAdapter | % {
 $InterfaceIndexArray += New-Object PSObject -property @{
 'index' = $_.InterfaceIndex;
 'Availability' = $_.Availability;
 'ConfigManagerErrorCode' = $_.ConfigManagerErrorCode;
 'Description' = $_.Description
 }
 }
 $CheckedInterface = $InterfaceIndexArray | ? { $_.index -eq $Interface }
 if ($CheckedInterface) {
 if ( ( $CheckedInterface.Availability -eq 3 ) -and ( $CheckedInterface.ConfigManagerErrorCode -eq 0 ) ) {
 Write-Debug "$($CheckedInterface.Description) status: OK. Adding the route"
 Start-Execute -setup $setup -param $param
 return $true
 }
 else {
 $ErrorAvailable = Get-ErrorCode -ErrorId $CheckedInterface.Availability -Availability
 $ErrorCode = Get-ErrorCode -ErrorId $CheckedInterface.ConfigManagerErrorCode -ConfigMngr
 Write-Error "$($CheckedInterface.Description) Availability: $ErrorAvailable - Error: $ErrorCode"
 return $false
 }
 }
 else {
 Write-Error "Network Interface $Interface doesn't exists."
 return $false
 }
 }
 else {
 Write-Error "Route.exe missing in C:\Windows\System32."
 return $false
 }
 }
 }
 Catch {
 Write-Error "Error Executing the remote command. Error: $($_.Exception.Message)"
 return $false
 }
 }
`

