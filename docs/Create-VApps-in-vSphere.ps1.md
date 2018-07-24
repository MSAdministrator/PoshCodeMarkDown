---
Author: ant b 2012
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 3237
Published Date: 2012-02-15t03
Archived Date: 2012-02-18t15
---

# create vapps in vsphere - 

## Description

a simple script to create vapps within vsphere

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 <#
 .SYNOPSIS
 
 Creates VApps within DevStack with reservations for CPU & RAM
 
 .DESCRIPTION
 
 Creates VApps within DevStack vSphere Environment, available parameters are:
 
 -VIServer (Optional, defaults to DevStack) {FQDN of VCentre Server}
 -AppName (Required) {VApp Label}
 -Location (Required) {Managed | Un-managed}
 -CPUReservation (Optional, defaults to 0) {Amount in MHz}
 -RAMReservation (Optional, defaults to 0) {Amount in MB}
 
 
 
 .EXAMPLE
 The example below creates a VApp named "Test VApp" in the Un-Managed Resource Pool with a RAM reservation of 512MB
 
 Create-VApp -AppName "Test VApp" -Location Un-managed -RAMReservation 512
 
 .EXAMPLE
 The example below shows the minimum required parameters, this VApp will be created with no reserved resources!
 
 Create-VApp -AppName "Test VApp1" -Location "Un-Managed"
 
 #>
 
 
 Param(
 [String]$VIServer = "<Your default VCentre Server>",
 [parameter(Mandatory=$true)][String]$AppName,
 [parameter(Mandatory=$true)][String]$Location,
 [Int]$CPURes,
 [Int]$RamRes
 )
     
 if ( (Get-PSSnapin -Name VMware.VimAutomation.Core -ErrorAction SilentlyContinue) -eq $null )
 {
     Add-PsSnapin VMware.VimAutomation.Core
 }
 
 Connect-VIServer $VIServer
 
 $Apps = Get-VApp | Select Name
 $i = 0
 ForEach ($App in $Apps)
 {
 If ($Apps[$i].Name -like $AppName)
 {
 Throw "There is already a VApp with the name $AppName"
 }
 Else{
 $i++
 }}
 
 New-vapp -Location $Location -Name $AppName -CpuExpandableReservation 0 -CpuReservationMhz $CPURes -CpuSharesLevel Normal -MemExpandableReservation 0 -MemReservationMB $RamRes -MemSharesLevel Normal
 
 Disconnect-VIServer -Server * -Force -Confirm:$False
`

