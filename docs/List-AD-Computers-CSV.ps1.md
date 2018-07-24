---
Author: alphasun
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2935
Published Date: 2012-08-31t14
Archived Date: 2015-05-09t23
---

# list ad computers csv - 

## Description

this script will list all computer objects (and some information about them) into a csv file. the �ping status� and various information items are determined through wmi. some filtering is done for special characters that regularly appear in the operating system caption entry and the hardware vendor entry. this is largely the same as the list ad computers xls script, but is intended for use on systems that do not have excel installed. the csv file that is produced can easily be imported into excel and re-saved as an xls at a later time if required.

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 $datetime = Get-Date -Format "yyMMdd-HHmm"
 
 $objDomain = New-Object System.DirectoryServices.DirectoryEntry
 [string]$DomainName = $objDomain.name
 $objSearcher = New-Object System.DirectoryServices.DirectorySearcher
 $objSearcher.SearchRoot = $objDomain
 $objSearcher.Filter = ("(objectCategory=computer)")
 $Results = $objSearcher.FindAll()
 
 $LogDir = "."
 $LogFile = "$DomainName-ComputerObjects-$datetime.csv"
 
 Add-Content "$LogDir\$LogFile" "UUID,Hostname,IP Address,Domain,OS,OS Version,OS Release,Manufacturer,Model,Platform,Memory(MB),Processors,Disk-Usage(KB)"
 
 Write-Host -ForegroundColor Black -BackgroundColor Yellow "Gathering information for all systems on the $DomainName domain"
 Write-Host -ForegroundColor Black -BackgroundColor Yellow "Results will be written to:" -noNewLine
 Write-Host " $LogDir\$LogFile" | Write-Host "" | Write-Host ""
 
 $WMI_CSP = "Win32_ComputerSystemProduct"
 $WMI_CS = "Win32_ComputerSystem"
 $WMI_Disk = "Win32_LogicalDisk"
 $WMI_OS = "Win32_OperatingSystem"
 $WMI_Proc = "Win32_Processor"
 
 foreach ($objResult in $Results)
 {
 	$objComputer = $objResult.Properties
 	$computer = $objComputer.name
 	$pingStatus = Get-WMIObject Win32_PingStatus -Filter "Address = '$computer'"
 	$ipAddress = $pingStatus.ProtocolAddress
 	if($pingStatus.StatusCode -eq 0)
 	{
 		$UUID = Get-WMIObject $WMI_CSP -ComputerName $computer | Select UUID
 		$_UUID = $UUID.UUID
 		$OS = Get-WMIObject $WMI_OS -ComputerName $computer  | Select Caption
 		$_OS = $OS.Caption
 		if($_OS)
 		{if($_OS.Contains(",")) {$_OS = $_OS.Replace(",","")} if($_OS.Contains("(R)")) {$_OS = $_OS.Replace("(R)","")}}
 		$CSDVer = Get-WMIObject $WMI_OS -ComputerName $computer  | Select CSDVersion
 		$_CSDVer = $CSDVer.CSDVersion
 		$Ver = Get-WMIObject $WMI_OS -ComputerName $computer  | Select Version
 		$_Ver = $Ver.Version
 		$Mfr = Get-WMIObject $WMI_CSP -ComputerName $computer  | Select Vendor
 		$_Mfr = $Mfr.Vendor
 		if($_Mfr)
 		{if($_Mfr.Contains("HP")) {$_Mfr = $_Mfr.Replace("HP","Hewlett-Packard")}}
 		$HW = Get-WMIObject $WMI_CSP -ComputerName $computer  | Select Name
 		$_HW = $HW.Name
 		$Platform = Get-WMIObject $WMI_Proc -ComputerName $computer -filter "DeviceID = 'CPU0'"  | Select Manufacturer
 		$_Platform = $Platform.Manufacturer
 		$RAMKB = Get-WMIObject $WMI_CS -ComputerName $computer  | Select TotalPhysicalMemory
 		[int64]$_RAMKB = $RAMKB.TotalPhysicalMemory
 		[int64]$_RAMMB = $_RAMKB / 1kb
 		$Processors = Get-WMIObject $WMI_CS -ComputerName $computer  | Select NumberOfProcessors
 		[int64]$NumOfProcs = $Processors.NumberOfProcessors
 		$Space = Get-WMIObject $WMI_Disk -ComputerName $computer -filter "DeviceID = 'c:'"  | Select FreeSpace
 		[int64]$DiskFree = $Space.FreeSpace
 		$Size = Get-WMIObject $WMI_Disk -ComputerName $computer -filter "DeviceID = 'c:'"  | Select Size
 		[int64]$DiskSize = $Size.Size
 		[int64]$DiskUsage = ($DiskSize - $DiskFree) / 1kb
 		Write-Host -ForegroundColor Green "Reply received from $computer ($ipAddress)"
 		Add-Content "$LogDir\$LogFile" "$_UUID,$computer,$ipAddress,$DomainName,$_OS,$_CSDVer,$_Ver,$_Mfr,$_HW,$_Platform,$_RAMMB,$NumOfProcs,$DiskUsage"
 	}
 	else
 	{
 		Write-Host -ForegroundColor Red "No Reply received from $computer ....................[SKIPPED]"
 		Add-Content "$LogDir\$LogFile" "HOST NOT ONLINE,$computer,---,$DomainName,---,---,---,---,---,---,---,---"
 	}
 }
`

