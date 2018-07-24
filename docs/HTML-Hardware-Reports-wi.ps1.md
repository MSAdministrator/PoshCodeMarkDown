---
Author: ranger69
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: utf-8
License: cc0
PoshCode ID: 4114
Published Date: 2013-04-18t05
Archived Date: 2013-04-24t08
---

# html hardware reports wi - 

## Description

author

## Comments



## Usage



## TODO



## function

`get-hardwarereport`

## Code

`#
 #
 Function Get-HardwareReport {
 [CmdletBinding()] Param
 (
 [parameter(Mandatory=$true,
 ValueFromPipeline=$true)] [String[]]$devices
 )
 Begin {Write-Host " Starting Hardware Reports"}
 Process {
 foreach ($device in $devices) {
 $name=$device
 $filepath="$home\$name.html"
 $PingDevice=Test-Connection $name -count 1 -quiet;trap{continue}
 If ($PingDevice -eq �True�) {
 Write-Host " Processing info for $name "
 $a = "<style>"
 $a = $a + "BODY{background-color:Lavender ;}"
 $a = $a + "TABLE{border-width: 1px;border-style: solid;border-color: black;border-collapse: collapse;}"
 $a = $a + "TH{border-width: 1px;padding: 0px;border-style: solid;border-color: black;background-color:skyblue}"
 $a = $a + "TD{border-width: 1px;padding: 0px;border-style: solid;border-color: black;background-color:lightyellow}"
 $a = $a + "</style><Title>$name Hardware Report</Title>"
 ####################################
 ConvertTo-Html -Head $a -Body "<h1> Computer Name : $name </h1>" > "$filepath"
 Get-WmiObject -ComputerName $name Win32_BaseBoard | Select Name,Manufacturer,Product,SerialNumber,Status | ConvertTo-html -Body "<H2> MotherBoard Information</H2>" >> "$filepath"
 Get-WmiObject win32_bios -ComputerName $name | Select Manufacturer,Name,@{name='biosversion' ; Expression={$_.biosversion -join '; '}}, @{name='ListOfLanguages' ; Expression={$_.ListOfLanguages -join '; '}},PrimaryBIOS,ReleaseDate,SMBIOSBIOSVersion,SMBIOSMajorVersion,SMBIOSMinorVersion | ConvertTo-html -Body "<H2> BIOS Information </H2>" >>"$filepath"
 Get-WmiObject Win32_CDROMDrive -ComputerName $name | select Name,Drive,MediaLoaded,MediaType,MfrAssignedRevisionLevel | ConvertTo-html -Body "<H2> CD ROM Information</H2>" >> "$filepath"
 Get-WmiObject Win32_ComputerSystemProduct -ComputerName $name | Select Vendor,Version,Name,IdentifyingNumber,UUID | ConvertTo-html -Body "<H2> System Information </H2>" >> "$filepath"
 Get-WmiObject win32_diskDrive -ComputerName $name | select Model,SerialNumber,InterfaceType,Size,Partitions | ConvertTo-html -Body "<H2> Harddisk Information </H2>" >> "$filepath"
 Get-WmiObject win32_networkadapter -ComputerName $name | Select Name,Manufacturer,Description ,AdapterType,Speed,MACAddress,NetConnectionID | ConvertTo-html -Body "<H2> Network Card Information</H2>" >> "$filepath"
 Get-WmiObject Win32_PhysicalMemory -ComputerName $name | select BankLabel,DeviceLocator,Capacity,Manufacturer,PartNumber,SerialNumber,Speed | ConvertTo-html -Body "<H2> Physical Memory Information</H2>" >> "$filepath"
 Get-WmiObject Win32_Processor -ComputerName $name | Select Name,Manufacturer,Caption,DeviceID,CurrentClockSpeed,CurrentVoltage,DataWidth,L2CacheSize,L3CacheSize,NumberOfCores,NumberOfLogicalProcessors,Status | ConvertTo-html -Body "<H2> CPU Information</H2>" >> "$filepath"
 Get-WmiObject Win32_SystemEnclosure -ComputerName $name | Select Tag,InstallDate,LockPresent,PartNumber,SerialNumber | ConvertTo-html -Body "<H2> System Enclosure Information </H2>" >> "$filepath"
 invoke-Expression "$filepath"
 }else {Write-Host " $name is offline - will not be included in the reports"}
 }
 }
 End {Write-Host "Ending Hardware Report for $name"}
 }
`

