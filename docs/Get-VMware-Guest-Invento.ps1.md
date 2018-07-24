---
Author: omarr
Publisher: 
Copyright: 
Email: 
Version: 1.5
Encoding: utf-8
License: cc0
PoshCode ID: 4737
Published Date: 2016-12-24t09
Archived Date: 2016-03-06t12
---

# get-vmware-guest-invento - 

## Description

this script will create an inventory of all guests in the target virtual center and then create a csv

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 <#>
 ==========================================================================
 
 This script was created and tested on the following:
 
 		Name                     Version
 		-----------------        -----------------
 		PowerShell               :2.0
 		VMWare PowerCLi          :4.1.1.0
 		Quest AcitveRoles for AD :1.4.0.2139
 
 ==============================
  NAME:   |Get-VMware-Guest-Inventory.ps1
 ==============================
  AUTHOR: |Omarr Miller
  EMAIL:  |omarr.miller@delta.com
 ==============================
 
 COMMENT: 
 		This script will create an inventory of all Guests within the 
 		requested Virtual Center and save it as a .csv file.
 		
 		If you need to inventory only a specific cluster, edit lines 54, 55 & 58
 		
 
  	You have a royalty-free right to use, modify, reproduce, and
  	distribute this script file in any way you find useful, provided that
  	you agree that the creator, owner above has no warranty, obligations,
  	or liability for such use.
 
 REQUIREMENTS OF THIS SCRIPT:
 		Name                     Version
 		-----------------        -----------------
 		PowerShell               :2.0
 		VMWare PowerCLi          :4.1.1.0
 		Quest AcitveRoles for AD :1.4.0.2139
 
 
 VERSION HISTORY:
 		1.0 3/22/2011 - Initial release
 		1.5 7/21/2011 - Updated Get-View syntax
 
 ==========================================================================
 </#>
 $VirtualCenter = Read-Host "10.1.100.86"
 #$ResourcePool = Read-Host "Please enter the complete name of the target Resource Pool"
 $FileLocation = Read-Host "D:\GuestList.csv"
 
 Connect-VIServer $VirtualCenter
 $stats = @()
 
 #$VMCluster = Read-Host "Please enter the name of the HA Cluster"
 #$ServerList = Get-VM -Location $VMCluster
 
 #$ServerList = Get-ResourcePool $ResourcePool | Get-VM
 #$ServerList = Get-VM | Where {$_.PowerState -match "on"} | Sort Name
 $ServerList = Get-VM | Sort Name
 Clear
 Foreach ($Guests in $ServerList) {
 	$Guest = $Guests.Name.ToUpper()
 	
 	$VM = Get-VM $Guest
 	#$VMGuest = Get-VMGuest $Guest
 	$ESXHost = $VM.Host.Name.ToUpper()
 	$VMHost = Get-VMHost $ESXHost
 	$row = New-Object System.Object
 	$row | Add-Member -Type NoteProperty -Name "Guest" -Value $Guest
 	$row | Add-Member -Type NoteProperty -Name "DNS Name" -Value $VM.ExtensionData.Guest.HostName.ToUpper()
 	$row | Add-Member -Type NoteProperty -Name "Power State" -Value $VM.PowerState
 	$row | Add-Member -Type NoteProperty -Name "Guest OS Full Name" -Value $VM.Guest.OSFullName
 	$row | Add-Member -Type NoteProperty -Name "Guest RAM (MB)" -Value $VM.MemoryMB
 	$row | Add-Member -Type NoteProperty -Name "Guest vCPU Count" -Value $VM.NumCPU
 	$row | Add-Member -Type NoteProperty -Name "Guest Hardware Ver" -Value $VM.Version
 	$row | Add-Member -Type NoteProperty -Name "Guest VMTools Status" -Value $VM.ExtensionData.Guest.ToolsStatus
 	$row | Add-Member -Type NoteProperty -Name "Guest VMTools Version" -Value $VM.ExtensionData.Guest.ToolsVersion
 	$row | Add-Member -Type NoteProperty -Name "Guest VMTools Version Status" -Value $VM.ExtensionData.Guest.ToolsVersionStatus
 	$row | Add-Member -Type NoteProperty -Name "Guest VMTools Running Status" -Value $VM.ExtensionData.Guest.ToolsRunningStatus
 	$row | Add-Member -Type NoteProperty -Name "Guest IP" -Value $VM.ExtensionData.Summary.Guest.IpAddress
 	$row | Add-Member -Type NoteProperty -Name "Guest Provisioned Space (GB)" -Value $VM.ProvisionedSpaceGB
 	$DiskCount = 0
 	$DT = @()
 	ForEach ($vDisk in $VM.Guest.Disks) {
 		$DriveLoc = "Guest Drive " + $DiskCount + " Location"
 		$DriveLetter = "Guest Drive " + $DiskCount + ""
 		$DriveSize = "Guest Drive " + $DiskCount + " Size"
 		$DriveFree = "Guest Drive " + $DiskCount + " Free Space"
 		$vDiskCap = [math]::Round(($vDisk.Capacity) / 1GB)
 		$vDiskFree = [math]::Round(($vDisk.FreeSpace) / 1GB)
 		$vDiskLoc = $VM.HardDisks[$DiskCount]
 		$row | Add-Member -Type NoteProperty -Name $DriveLetter -Value $vDisk.Path
 		$row | Add-Member -Type NoteProperty -Name $DriveSize -Value $vDiskCap
 		$row | Add-Member -Type NoteProperty -Name $DriveFree -Value $vDiskFree
 		$row | Add-Member -Type NoteProperty -Name $DriveLoc -Value $vDiskLoc.Filename
 		$DiskCount++
 		$DriveTotals = "" + $row.$DriveLetter + " " + $row.$DriveSize + ";"
 		$DT += $DriveTotals
 		}
 	$row | Add-Member -Type NoteProperty -Name "Host Name" -Value $VMHost.ExtensionData.Summary.Config.Name.ToUpper()
 	$ESXCluster = Get-Cluster -VMHost $ESXHost -ErrorAction SilentlyContinue
 	IF (!$ESXCluster)
 	{
 		$row | Add-Member -Type NoteProperty -Name "Host is Member of Cluster" -Value "Stand Alone Host"
 	} ELSE {
 		$row | Add-Member -Type NoteProperty -Name "Host is Member of Cluster" -Value $ESXCluster.Name.ToUpper()
 	}
 	$row | Add-Member -Type NoteProperty -Name "Host Vendor" -Value $VMHost.ExtensionData.Hardware.SystemInfo.Vendor
 	$row | Add-Member -Type NoteProperty -Name "Host Model" -Value $VMHost.ExtensionData.Hardware.SystemInfo.Model
 	$HostRam = [math]::Round(($VMHost.ExtensionData.Summary.Hardware.MemorySize) / 1GB)
 	$row | Add-Member -Type NoteProperty -Name "Host RAM" -Value $HostRam
 	$row | Add-Member -Type NoteProperty -Name "Host CPU Model" -Value $VMHost.ExtensionData.Summary.Hardware.CpuModel
 	$row | Add-Member -Type NoteProperty -Name "Host CPU Count" -Value $VMHost.ExtensionData.Summary.Hardware.NumCpuThreads
 	$row | Add-Member -Type NoteProperty -Name "Host CPU Speed" -Value $VMHost.ExtensionData.Summary.Hardware.CpuMhz
 	$row | Add-Member -Type NoteProperty -Name "Host Product Name" -Value $VMHost.ExtensionData.Summary.Config.Product.Name
 	$row | Add-Member -Type NoteProperty -Name "Host Product Version" -Value $VMHost.ExtensionData.Summary.Config.Product.Version
 	$row | Add-Member -Type NoteProperty -Name "Host Product Build" -Value $VMHost.ExtensionData.Summary.Config.Product.Build
 	$row | Add-Member -Type NoteProperty -Name "Host Service Console" -Value $VMHost.ExtensionData.Config.Network.vNic[0].Spec.IP.IPAddress
 	$row | Add-Member -Type NoteProperty -Name "Host Service Console Subnet Mask" -Value $VMHost.ExtensionData.Config.Network.vNic[0].Spec.IP.SubnetMask
 	$row | Add-Member -Type NoteProperty -Name "Host Service Console 1" -Value $VMHost.ExtensionData.Config.Network.vNic[1].Spec.IP.IPAddress
 	$row | Add-Member -Type NoteProperty -Name "Host Service Console 1 Subnet Mask" -Value $VMHost.ExtensionData.Config.Network.vNic[1].Spec.IP.SubnetMask
 	$row | Add-Member -Type NoteProperty -Name "Host vMotion IP Address" -Value $VMHost.ExtensionData.Config.vMotion.IPConfig.IpAddress
 	$row | Add-Member -Type NoteProperty -Name "Host vMotion Subnet Mask" -Value $VMHost.ExtensionData.Config.vMotion.IPConfig.SubnetMask
 	$row
 	$stats += $row	
 }
 $stats | Export-Csv -Force .\$FileLocation -NoTypeInformation
 Invoke-Item .\$FileLocation
`

