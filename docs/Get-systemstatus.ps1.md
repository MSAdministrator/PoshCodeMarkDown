---
Author: patrickg
Publisher: 
Copyright: 
Email: 
Version: 802.11
Encoding: utf-8
License: cc0
PoshCode ID: 6460
Published Date: 2016-07-30t21
Archived Date: 2016-10-30t01
---

# get-systemstatus - 

## Description

my script uses wmi to gather various useful pieces of useful systems information that can be used for either support calls, downloading drives or troubleshooting your system.

## Comments



## Usage



## TODO



## script

`get-systemstatus`

## Code

`#
 #
 <#
 .Synopsis
    This script is to give you some basic info for troubleshooting or gathering data for a support call
 .DESCRIPTION
    The output is broken down by system information, Networking info, Anti Virus Products, and drive info auto startup programs.
    Patrick G 7/30/2016
 
 .EXAMPLE 
    No command line arguments
 .OUTPUTS
 ========== Networking Info ==========
 NIC: Dell Wireless 1705 802.11b/g/n (2.4GHZ)
 DHCP Enabled: True
 IP Address: 172.168.1.65
 Default gateway: 172.168.1.254
 
 
 ========== Anti Virus Products and Status========== 
 Kaspersky Internet Security
 Definition status:  Up to date
 Real-time protection status: Enabled
 Windows Defender
 Definition status:  Up to date
 Real-time protection status: Disabled
 Defender Service is not running
 
 
 ========== System Information==========
 Serial Number: 88X0T32
 Bios:DELL   - 1072009 A08 American Megatrends - 4028E
 Computer Model: Inspiron 3847
 System Type: x64-based PC
 Number of Physical Processors: 1
 Operating System: Microsoft Windows 10 Home
 Operating System Build Number: 10586
 last reboot Time: 07/27/2016 13:21
 
 
 ========== Drive Information ==========
 Drive Letter: C:
 Freespace: 831 GB
 Total Drive Capacity 917 GB
 Percent free: 91%
 
 ========== Auto Start Programs ==========
 Send to OneNote
 OneDrive
 Lync
 kpm.exe
 Secunia PSI Tray
 RTHDVCPL
 RtHDVBg
 IAStorIcon
 WavesSvc
 iTunesHelper
 
    
 #>
 function get-systemstatus
 {
    [CmdletBinding()]
   
     Param
     (
         
     )
 
     
    
     }
 
 
 Function Get-startups
 
 {
 Write-Output `n "========== Auto Start Programs =========="
 
 Get-CimInstance Win32_startupCommand | select -ExpandProperty name
 }
 
 
 
 function get-driveinfo
 {
     
 
     $Drvinfo = Get-WmiObject -Query �SELECT * from win32_logicaldisk where DriveType = '3'�
     foreach($Drives in $Drvinfo)
     {
         if ($Drives.size -gt 1073741823)
         {
         Write-Output `n
         Write-Output "========== Drive Information =========="
         $DrvLetter= $Drives.DeviceID
         Write-Output "Drive Letter: $DrvLetter"
         $Drvfree = $Drives.Freespace
         $FormatedDrvfree = $Drvfree/1gb 
         $FormatedDrvfree ="{0:N0}" -f $FormatedDrvfree
         Write-Output "Freespace: $FormatedDrvfree GB"
         $DrvSize = $Drives.size
         $FormatedDrvSize = $DrvSize/1gb 
         $FormatedDrvSize  ="{0:N0}" -f $FormatedDrvSize 
         Write-Output "Total Drive Capacity $FormatedDrvSize GB"
         $PerFree = $FormatedDrvfree/ $FormatedDrvSize * 100
         $PerFree  ="{0:N0}" -f $PerFree
         Write-Output "Percent free: $PerFree%"
         }
     }
 }
 
 
 
 function Get-IP 
 {
 
 
 $NicInfo = Get-WmiObject -Namespace root\CIMV2 -Class Win32_NetworkAdapterConfiguration
 
 Write-output "========== Networking Info =========="
 
     foreach ($nics in $NicInfo)
     {
 
         if ($nics.IPAddress -ne $Null)
             {
             $NICDesc = $nics.description
             Write-Output "NIC: $NICDesc"
             $Dhcpstat =  $nics.DHCPENABLED
             write-output "DHCP Enabled: $Dhcpstat"
             $IPinfo = $nics.IPADDRESS[0]
             Write-Output "IP Address: $IPinfo"
             $DG = $nics.DefaultIPGateway
             Write-Output "Default gateway: $DG"
             }
     }
 
 
     write-Output `n
 
 
 }
 
 
 function Get-Av   {
 
 $AntiVirusProduct = Get-WmiObject -Namespace root\SecurityCenter2 -Class AntiVirusProduct 
 
 Write-Output "========== Anti Virus Products and Status========== "
 
 foreach($avp in $AntiVirusProduct)
 {
 
 $avp.displayname
 
 switch ($avp.productState) { 
     "262144" {$defstatus = "Up to date" ;$rtstatus = "Disabled"} 
     "262160" {$defstatus = "Out of date" ;$rtstatus = "Disabled"} 
     "266240" {$defstatus = "Up to date" ;$rtstatus = "Enabled"} 
     "266256" {$defstatus = "Out of date" ;$rtstatus = "Enabled"} 
     "393216" {$defstatus = "Up to date" ;$rtstatus = "Disabled"} 
     "393232" {$defstatus = "Out of date" ;$rtstatus = "Disabled"} 
     "393488" {$defstatus = "Out of date" ;$rtstatus = "Disabled"} 
     "397312" {$defstatus = "Up to date" ;$rtstatus = "Enabled"} 
     "397328" {$defstatus = "Out of date" ;$rtstatus = "Enabled"} 
     "397584" {$defstatus = "Out of date" ;$rtstatus = "Enabled"} 
     "397568" {$defstatus = "Up to date"; $rtstatus = "Enabled"}
     "393472" {$defstatus = "Up to date" ;$rtstatus = "Disabled"}
     default {$defstatus = "Unknown" ;$rtstatus = "Unknown"} 
 }
 Write-Output "Definition status:  $defstatus"
 
 Write-output "Real-time protection status: $rtstatus" 
 
 }
 $Defstat = Get-Service windefend
 if ($Defstat.Status -match "stop" )
 {
 
     Write-output "Defender Service is not running" 
 
 }
 
 }
 
 
 
 
 function Get-sysinformation
 {
     Write-Output "`n"
     Write-Output "========== System Information=========="
     $sysdata = gwmi win32_systemenclosure 
     $SerNum = $sysdata.SerialNumber 
     Write-Output "Serial Number: $SerNum" 
 
     $bos= gwmi win32_bios 
     
     $BMfg  = $bos.Manufacturer 
     $Bv =  $bos.BIOSVersion
     write-host "Bios:$BV"
 
     $coms = Get-WmiObject -Class:Win32_ComputerSystem
     $CMPMod =  $coms.Model
     Write-Output  "Computer Model: $CMPMod"
 
     $SysTYPE = $coms.SystemType
     Write-Output "System Type: $SysTYPE"
     
     $Sysprocs= $coms.NumberOfProcessors
     Write-Output "Number of Physical Processors: $Sysprocs"
     
     $OSdata = gwmi Win32_OperatingSystem
 
 
     $OSName= $OSdata.CAPTION
     Write-Output "Operating System: $OSName"
     
     $OSBld = $OSdata.BuildNumber
     Write-Output "Operating System Build Number: $OSBld"
 
     #$OSdata.OSArchitecture
     #$OSdata.Properties
     
     $YrLBUP = $OSdata.lastBootUpTime[0] + $OSdata.lastBootUpTime[1] + $OSdata.lastBootUpTime[2] + $OSdata.lastBootUpTime[3]
     $MnthLBUP = $OSdata.lastBootUpTime[4] + $OSdata.lastBootUpTime[5]
     $DayLBUP = $OSdata.lastBootUpTime[6] + $OSdata.lastBootUpTime[7]
     $HRLBUP = $OSdata.lastBootUpTime[8] + $OSdata.lastBootUpTime[9] + ':'
     $MinLBUP = $OSdata.lastBootUpTime[10] + $OSdata.lastBootUpTime[11] 
 
     write-output "last reboot Time: $MnthLBUP/$DayLBUP/$YrLBUP $HRLBUP$MinLBUP"
 
 
     }
    
  
   
 
     Get-IP
     Get-av
     Get-sysinformation
     get-driveinfo
     Get-startups
`

