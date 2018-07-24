---
Author: rich kusak
Publisher: 
Copyright: 
Email: 
Version: 6.1
Encoding: ascii
License: cc0
PoshCode ID: 1451
Published Date: 2010-11-03t07
Archived Date: 2016-09-28t06
---

# vhdfunctions.psm1 - 

## Description

here’s several functions for working with vhd’s in windows 7 and windows server 2008 r2. i’ve been working with powershell for about a year and this is my first go at a module. i’m a sysadmin and not a developer so some of my solutions are in that mode of thinking. there’s probably .net ways to accomplish what i did and i’m certainly open to learning if there’s a better way. i’ve found these functions useful and hopefully someone else out there will too. enjoy.

## Comments



## Usage



## TODO



## function

`dismount-vhd`

## Code

`#
 #
 <#
 
 Name: VHDFunctions.psm1
 Author: Rich Kusak (rkusak@cbcag.edu)
 Created: 2009-10-23
 LastEdit: 2009-11-03 08:57
 
 Included Functions:
 	Dismount-VHD
 	Initialize-VHD
 	Mount-VHD
 	New-VHD
 	Set-VHDBootConfiguration
 	Test-VHD
 	
 #>
 
 <#
 .SYNOPSIS
 	Dismount a VHD file from the system.
 
 .DESCRIPTION
 	This function wraps the consistancy of PowerShell around the Diskpart utility.
 	A Diskpart script is created to automate the dismount (detach) of a VHD file from the system.
 	Optionally, the VHD file can be deleted following detachment.
 	
 .PARAMETER Path
 	Specifies the full path to the VHD file.
 
 .PARAMETER Remove
 	Removes (deletes) the VHD file after dismounting it.
 
 .PARAMETER NoConfirm
 	Supresses the delete confirmation prompt.
 
 .PARAMETER DiskpartScript
 	Specifies the path location of the Diskpart script file.
 	Default location is $env:SystemDrive
 	This file is deleted at the conclusion of the script.
 
 .PARAMETER Rescan
 	Instructs Diskpart to rescan the system for available storage resources.
 	
 .EXAMPLE
 	Dismount-VHD -Path C:\test.vhd
 	Dismounts the specified VHD file.
 	
 .EXAMPLE
 	Dismount-VHD -Path C:\test.vhd -Remove
 	Dismounts the specified VHD file and then deletes it.
 
 .NOTES
 	Name: Dismount-VHD.ps1
 	Author: Rich Kusak
 	Created: 2009-10-22
 	LastEdit: 2009-10-26 11:35
 	
 #>
 function Dismount-VHD {
 	[CmdletBinding()]
 	param (
 		[Parameter(Position=0,Mandatory=$true,ValueFromPipeline=$false)]
 			[string]$Path,
 		[switch]$Remove,
 		[switch]$NoConfirm,
 		[string]$DiskpartScript = "$env:SystemDrive\DiskpartScript.txt",
 		[switch]$Rescan
 	)
 	
 	begin {
 		function InvokeDiskpart {
 			Diskpart.exe /s $DiskpartScript
 		}
 		
 		function RemoveVHD {
 			switch ($NoConfirm) {
 				$false {
 					"" ; Write-Warning "Are you sure you want to delete the file ""$Path""?"
 					$Prompt = Read-Host "Type ""YES"" to continue or anything else to break"
 					if ($Prompt -ceq 'YES') {
 						Remove-Item -Path $Path -Force
 						"" ; Write-Host "VHD ""$Path"" deleted!" ; ""
 					} else {
 						"" ; Write-Host "Script terminated without deleting the VHD file." ; ""
 					}
 				}
 				$true {
 					Remove-Item -Path $Path -Force
 					"" ; Write-Host "VHD ""$Path"" deleted!" ; ""
 				}
 			}
 		}
 		if (Get-WmiObject win32_OperatingSystem -Filter "Version < '6.1'") {throw "The script operation requires at least Windows 7 or Windows Server 2008 R2."}
 	}
 	process{
 @"
 $(if ($Rescan) {'Rescan'})
 Select VDisk File="$Path"`nDetach VDisk
 Exit 
 "@ | Out-File -FilePath $DiskpartScript -Encoding ASCII -Force
 		InvokeDiskpart
 	}
 	end {
 		if ($Remove) {RemoveVHD}
 		Remove-Item -Path $DiskpartScript -Force ; ""
 	}
 }
 Export-ModuleMember -Function Dismount-VHD
 
 
 <#
 .SYNOPSIS
 	Initialize a VHD by preparing it for use.
 
 .DESCRIPTION
 	This function wraps the consistancy of PowerShell around the Diskpart utility.
 	A Diskpart script is created to automate initializing a VHD.
 	The script creates a partition, assigns a drive letter, and formats a mounted VHD.
 	
 .PARAMETER Path
 	Specifies the full path to the VHD file.
 
 .PARAMETER Drive
 	A drive letter to assign to the mounted VHD.
 	If not specified the system will auto assign the next available drive letter.
 
 .PARAMETER Label
 	A volume label to assign to the mounted VHD.
 
 .PARAMETER DiskpartScript
 	Specifies the path location of the Diskpart script file.
 	Default location is $env:SystemDrive
 	This file is deleted at the conclusion of the script.
 
 .PARAMETER Rescan
 	Instructs Diskpart to rescan the system for available storage resources.
 
 .EXAMPLE
 	Initialize-VHD C:\test.vhd X: TestVHD
 	Initializes the VHD at path C:\test.vhd assign it to drive letter X: and give it the volume label "TestVHD".
 
 .NOTES
 	Name: Initialize-VHD
 	Author:	Rich Kusak
 	Created: 2009-10-22
 	LastEdit: 2009-10-26 15:11
 	
 #>
 function Initialize-VHD {
 	[CmdletBinding()]
 	param (
 		[Parameter(Position=0,Mandatory=$true,ValueFromPipeline=$false)]
 			[string]$Path,
 		[Parameter(Position=1,Mandatory=$false,ValueFromPipeline=$false)]
 			[string]$Drive,
 		[Parameter(Position=2,Mandatory=$false,ValueFromPipeline=$false)]
 			[string]$Label,
 		[Parameter(Position=3,Mandatory=$false,ValueFromPipeline=$false)]
 			[string]$DiskpartScript = "$env:SystemDrive\DiskpartScript.txt",
 		[switch]$NoConfirm,
 		[switch]$Rescan
 	)
 	
 	begin {
 		function InvokeDiskpart {
 			Diskpart.exe /s $DiskpartScript
 		}
 	
 		if (Get-WmiObject win32_OperatingSystem -Filter "Version < '6.1'") {throw "The script operation requires at least Windows 7 or Windows Server 2008 R2."}
 		if ($Drive) {
 			$Reserved = @('A:','B:','C:')
 			$Reserved += (Get-WmiObject win32_LogicalDisk -Property DeviceID | ForEach-Object {$_.DeviceID})
 			switch ($Drive) {
 				{($_ -notmatch "^[a-z]$") -and ($_ -notmatch "^[a-z]:$")} {throw "The drive letter ""$_"" is invalid."}
 				{$_ -notmatch ":"} {$Drive += ":"}
 				{$Reserved -contains $Drive} {throw "The drive letter ""$_"" is reserved."}
 			}
 		}
 		if (!$NoConfirm) {
 			"" ; Write-Warning "The VHD ""$Path"" is about to initialized. Any existing data will be destroyed!`nAre you sure you want to continue?" ; ""
 			$Prompt = Read-Host "Type ""YES"" to continue or anything else to break"
 			if ($Prompt -cne 'YES') {Write-Host "Function terminated by user."; "" ; break}
 		}
 	}
 	process {
 @"
 $(if ($Rescan) {'Rescan'})
 Select VDisk File="$Path"
 Clean
 Create Partition Primary
 Format Quick FS=NTFS $(if ($Label) {"Label=""$Label"""})
 $(if ($Drive) {"Assign Letter=$Drive"} else {'Assign'})
 Detail VDisk
 Exit
 "@ | Out-File -FilePath $DiskpartScript -Encoding ASCII -Force
 	}
 	end {
 		InvokeDiskpart
 		Remove-Item -Path $DiskpartScript -Force ; ""
 		Write-Host "The VHD ""$Path"" has been successfully initialized." ; ""
 	}
 }
 Export-ModuleMember -Function Initialize-VHD
 
 
 <#
 .SYNOPSIS
 	Mount a VHD to the system.
 
 .DESCRIPTION
 	This function wraps the consistancy of PowerShell around the Diskpart utility.
 	A Diskpart script is created to automate mounting (attach) a VHD file to the system.
 	
 .PARAMETER Path
 	Specifies the full path to the VHD file.
 	
 .PARAMETER DiskpartScript
 	Specifies the path location of the Diskpart script file.
 	Default location is $env:SystemDrive
 	This file is deleted at the conclusion of the script.
 
 .PARAMETER Rescan
 	Instructs Diskpart to rescan the system for available storage resources.
 
 .NOTES
 	Name: Mount-VHD.ps1
 	Author: Rich Kusak
 	Created: 2009-10-22
 	LastEdit: 2009-10-26 09:25
 	
 #>
 function Mount-VHD {
 	[CmdletBinding()]
 	param (
 		[Parameter(Position=0,Mandatory=$true,ValueFromPipeline=$false)]
 			[string]$Path,
 		[string]$DiskpartScript = "$env:SystemDrive\DiskpartScript.txt",
 		[switch]$Rescan
 	)
 	
 	begin {
 		function InvokeDiskpart {
 			Diskpart.exe /s $DiskpartScript
 		}
 		if (Get-WmiObject win32_OperatingSystem -Filter "Version < '6.1'") {throw "The script operation requires at least Windows 7 or Windows Server 2008 R2."}
 	}
 	process{
 @"
 $(if ($Rescan) {'Rescan'})
 Select VDisk File="$Path"`nAttach VDisk
 Exit 
 "@ | Out-File -FilePath $DiskpartScript -Encoding ASCII -Force
 		InvokeDiskpart
 	}
 	end {
 		Remove-Item -Path $DiskpartScript -Force ; ""
 		Write-Host "The VHD ""$Path"" has been successfully mounted." ; ""
 	}
 }
 Export-ModuleMember -Function Mount-VHD
 
 
 <#
 .SYNOPSIS
 	Create a new VHD file.
 
 .DESCRIPTION
 	This function wraps the consistancy of PowerShell around the Diskpart utility.
 	A Diskpart script is created to automate the creation of the VHD.
 	Optionally, the VHD can be mounted immediately following the creation process.
 	
 .PARAMETER Path
 	Specifies the full path to the VHD file.
 	
 .PARAMETER Maximum
 	The maximum space allocated for the VHD to use.
 
 .PARAMETER Fixed
 	Creates a fixed disk VHD file. By default a dynamically expanding VHD file is created.
 	
 .PARAMETER Mount
 	Mount (attach) the VHD to the system making it available to Windows.
 
 .PARAMETER Prepare
 	Prepares the VHD for use by partitioning and mounting (attach) the VHD to the system making it available to Windows.
 
 .PARAMETER NoConfirm
 	Supresses the maximum validation warning confirmation prompt.
 	
 .PARAMETER DiskpartScript
 	Specifies the path location of the Diskpart script file.
 	Default location is $env:SystemDrive
 	This file is deleted at the conclusion of the script.
 
 .PARAMETER Rescan
 	Instructs Diskpart to rescan the system for available storage resources.
 
 .NOTES
 	Name: New-VHD
 	Author: Rich Kusak
 	Created: 2009-05-27
 	LastEdit: 2009-10-26 10:06
 	
 #>
 function New-VHD {
 	[CmdletBinding()]
 	param (
 		[Parameter(Position=0,Mandatory=$true,ValueFromPipeline=$false)]
 			[string]$Path,
 		[Parameter(Position=1,Mandatory=$true,ValueFromPipeline=$false)]
 			[int]$Maximum,
 		[switch]$Fixed,
 		[switch]$Mount,
 		[switch]$NoConfirm,
 		[string]$DiskpartScript = "$env:SystemDrive\DiskpartScript.txt",
 		[switch]$Rescan
 	)
 	
 	begin {
 		function InvokeDiskpart {
 			Diskpart.exe /s $DiskpartScript
 		}
 		function TestMaximum {
 			$Drive = Split-Path $Path -Qualifier
 			$LogicalDisk = Get-WmiObject win32_LogicalDisk -Filter "DeviceID = '$Drive'"
 			$FreeSpace = [math]::Truncate(($LogicalDisk.FreeSpace)/1MB)
 			$Percent = [math]::Round(($Maximum/$FreeSpace)*100,0)
 			switch ($Maximum) {
 				{$_ -gt 2088960} {throw "The -Maximum parameter value ""$Maximum"" exceeds the allowable VHD size."}
 				{$_ -ge $FreeSpace -and $Fixed} {throw "The -Maximum parameter value ""$Maximum"" exceeds available disk space."}
 				{$_ -ge $FreeSpace} {
 					"" ; Write-Warning "The VHD maximum size exceeds available disk space."
 					if (!$NoConfirm) {
 						Write-Host "Are you sure you want to continue?"
 						$Prompt = Read-Host "Type ""YES"" to continue or anything else to break"
 						if ($Prompt -cne 'YES') {
 							"" ; Write-Host "Script terminiated by user." ; ""
 							return
 						}
 					}
 				}
 				{$Percent -ge 80} {
 					"" ; Write-Warning "The VHD maximum size is $Percent% of the available disk space."
 					if (!$NoConfirm) {
 						Write-Host "Are you sure you want to continue?"
 						$Prompt = Read-Host "Type ""YES"" to continue or anything else to break"
 						if ($Prompt -cne 'YES') {
 							"" ; Write-Host "Script terminiated by user." ; ""
 							return
 						}
 					}
 				}
 			}
 			if (Get-WmiObject win32_OperatingSystem -Filter "Version < '6.1'") {throw "The script operation requires at least Windows 7 or Windows Server 2008 R2."}
 		}
 	}
 	process {
 		TestMaximum
 @"
 $(if ($Rescan) {'Rescan'})
 Create VDisk File="$Path" Maximum=$Maximum $(if ($Fixed) {'Type=Fixed'} else {'Type=Expandable'})
 $(if ($Mount) {"Select VDisk File=""$Path""`nAttach VDisk"})
 Exit 
 "@ | Out-File -FilePath $DiskpartScript -Encoding ASCII -Force
 		InvokeDiskpart
 	}
 	end {
 		Remove-Item -Path $DiskpartScript -Force ; ""
 		Write-Host "A new VHD has been created at ""$Path""." ; ""
 		if ($Mount) {Write-Host "The VHD has been successfully mounted." ; ""}
 	}
 }
 Export-ModuleMember -Function New-VHD
 
 
 <#
 .SYNOPSIS
 	Set the system boot configuration to boot from a VHD.
 
 .DESCRIPTION
 	This function wraps the consistancy of PowerShell around the BCDEdit tool.
 	The Boot Configuration DataStore Editor (BCDEdit) is used to set the necessary
 	boot configuration entry to optionally boot to a VHD during startup.
 	
 .PARAMETER Path
 	Specifies the full path to the VHD file.
 
 .PARAMETER Description
 	Description for the boot configuration entry.
 	
 .EXAMPLE
 	Set-VHDBootConfiguration	
 
 .NOTES
 	Name: Set-VHDBootConfiguration
 	Author: Rich Kusak
 	Created: 2009-10-22
 	LastEdit: 2009-10-26 10:14
 	
 #>
 function Set-VHDBootConfiguration {
 	[CmdletBinding()]
 	param (
 		[Parameter(Position=0,Mandatory=$true)]
 			[string]$Path,
 		[Parameter(Position=1,Mandatory=$true)]
 			[string]$Description
 	)
 	
 	begin {
 		if (!(Test-Path -Path $Path -Include "*.vhd")) {throw "The path ""$Path"" is invalid or does not contain a VHD file."}
 		$Drive = Split-Path -Path $Path -Qualifier
 		$UnQualifiedPath = Split-Path -Path $Path -NoQualifier
 	}
 	process {
 		$Copy = bcdedit /copy '{current}' /d $Description
 		$CLSID = $Copy | ForEach-Object {$_.Remove(0,37).Replace(".","")} 
 		bcdedit /set $CLSID device vhd=[$Drive]""$UnQualifiedPath""
 		bcdedit /set $CLSID osdevice vhd=[$Drive]""$UnQualifiedPath""
 		bcdedit /set $CLSID detecthal on
 		New-Object PSObject | Add-Member -MemberType NoteProperty -Name 'Identifier' -Value $CLSID -PassThru |
 	}
 	end {
 		Write-Host "The VHD ""$Path"" has been prepared for boot operation." ; ""
 	}
 }
 Export-ModuleMember -Function Set-VHDBootConfiguration
 
 
 <#
 .SYNOPSIS
 	Test a VHD.
 
 .DESCRIPTION
 	This script wraps the consistancy of PowerShell around the Diskpart utility.
 	A Diskpart script is created to return the details of the VHD (VDisk).
 	
 .PARAMETER Path
 	Specifies the full path to the VHD file.
 
 .PARAMETER DiskpartScript
 	Specifies the path location of the Diskpart script file.
 	Default location is $env:SystemDrive
 	This file is deleted at the conclusion of the script.
 
 .PARAMETER Rescan
 	Instructs Diskpart to rescan the system for available storage resources.
 
 .NOTES
 	Name: Test-VHD.ps1
 	Author: Rich Kusak
 	Created: 2009-10-23
 	LastEdit: 2009-11-03 08:55
 	
 #>
 function Test-VHD {
 	[CmdletBinding()]
 	param (
 		[Parameter(Position=0,Mandatory=$true,ValueFromPipeline=$false)]
 			[string]$Path,
 		[Parameter(Position=1,Mandatory=$false,ValueFromPipeline=$false)]
 			[string]$DiskpartScript = "$env:SystemDrive\DiskpartScript.txt",
 		[switch]$Rescan
 	)
 
 	begin {
 		function InvokeDiskpart {
 			Diskpart.exe /s $DiskpartScript
 		}
 		if (Get-WmiObject win32_OperatingSystem -Filter "Version < '6.1'") {throw "The script operation requires at least Windows 7 or Windows Server 2008 R2."}
 		if (!(Test-Path -Path $Path -Include "*.vhd")) {throw "The path ""$Path"" is not valid or does not contain a VHD file."}
 	}
 	process {
 @"
 Select VDisk File="$Path"
 Detail VDisk
 "@ | Out-File -FilePath $DiskpartScript -Encoding ASCII -Force
 		InvokeDiskpart
 	}
 	end {
 		Remove-Item -Path $DiskpartScript -Force ; ""
 	}
 }
 Export-ModuleMember -Function Test-VHD
`

