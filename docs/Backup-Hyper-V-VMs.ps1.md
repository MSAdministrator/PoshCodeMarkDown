---
Author: jhjbhjb h
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6430
Published Date: 2016-07-01t04
Archived Date: 2016-07-06t18
---

# backup hyper-v vms - 

## Description

this complete module greatly facilitates backing up your hyper-v virtual machines.

## Comments



## Usage



## TODO



## script

`backup-vms`

## Code

`#
 #
 
 <#
 		Directions for use:
 		Import this script using the Import-Module cmdlet
 
 		All output is logged to the backup directory in the $($BackupDriveLetter):\VMBackup\Backup-VMs.log file
 
 		Use the Backup-VMs cmdlet to begin the process
 			Parameter BackupDriveLetter indicates the drive to put this backup onto. It must be mounted to the host running the script.
 			Parameter VMHost defines the host that contains the VMs you want to back up. If it's blank, then it just targets the host the script is running on
 			Parameter VMNames defines the specific VMs you wish to backup, otherwise it'll back up all of them on the target host
 			Switch parameter ShutHostDownWhenFinished will cause the specified host (and any VMs running on it) to shut down upon completion of the backup
 
 		Example:
 		PS> Import-Module D:\Backup-VMs.ps1
 		PS> Backup-VMs -BackupDriveLetter F -VMHost HyperVHost -VMNames mydevmachine,broker77
 
 		----------------------------------------------------------------------------
 		Note that this script requires administrator privileges for proper execution
 		----------------------------------------------------------------------------
 
 		Note that this script requires the following:
 
 		PowerShell Management Library for Hyper-V (for the Get-VM and Export-VM cmdlets)
 		This installs itself wherever you downloaded it - make sure the HyperV folder finds its way to somewhere in $env:PSModulePath
 		http://pshyperv.codeplex.com/downloads/get/219013
 
 		Windows PowerShell Pack (for the Copy-ToZip cmdlet)
 		This installs to $home\Documents\WindowsPowerShell\Modules, make sure that this path is in $env:PSModulePath
 		http://archive.msdn.microsoft.com/PowerShellPack/Release/ProjectReleases.aspx?ReleaseId=3341
 #>
 
 $Logfile = ''
 
 function Backup-VMs {
 	<#
 			.SYNOPSIS
 			The main function
 	
 			.DESCRIPTION
 			The main function
 	
 			.PARAMETER BackupDriveLetter
 			A description of the BackupDriveLetter parameter.
 	
 			.PARAMETER VMHost
 			$BackupDriveLetter:\VMBackups\$backupDate
 	
 			.PARAMETER VMNames
 			the host that holds the vms we wish to back up, otherwise the one running the script
 	
 			.PARAMETER ShutHostDownWhenFinished
 			if not specified, back up all of them
 	
 			.NOTES
 			My Version of Backup Hyper-V VMs by jhjbhjb h
 	#>
 	
 	[CmdletBinding(SupportsShouldProcess = $true)]
 	param
 	(
 		[Parameter(Mandatory = $true)]
 		[System.String]$BackupDriveLetter,
 		[ValidateNotNullOrEmpty()]
 		[System.String]$VMHost,
 		[System.String[]]$VMNames,
 		[switch]$ShutHostDownWhenFinished
 	)
 	
 	process {
 		$isPowerShellPackLoaded = Get-Module -Name PowerShellPack
 		if (!$isPowerShellPackLoaded) {
 			Write-Host -Object 'Attempting to load PowerShellPack modules...'
 			Import-Module -Name PowerShellPack
 			$isPowerShellPackLoaded = Get-Module -Name PowerShellPack
 			if (!$isPowerShellPackLoaded) {
 				Write-Host -ForegroundColor Red -Object 'Cannot load PowerShellPack module - terminating backup script.'
 				Break
 			}
 		}
 		$isHyperVModuleLoaded = Get-Module -Name HyperV
 		if (!$isHyperVModuleLoaded) {
 			Write-Host -Object 'Attempting to load HyperV module...'
 			Import-Module -Name HyperV
 			$isHyperVModuleLoaded = Get-Module -Name HyperV
 			if (!$isHyperVModuleLoaded) {
 				Write-Host -ForegroundColor Red -Object 'Cannot load HyperV module - terminating backup script.'
 				Break
 			}
 		}
 		if ($BackupDriveLetter -like '*:') {
 			$BackupDriveLetter = $BackupDriveLetter -replace ".$"
 		}
 		if ((Test-Path -Path "$($BackupDriveLetter):") -eq $false) {
 			Write-Host -ForegroundColor Red -Object "Drive $($BackupDriveLetter): does not exist - terminating backup script."
 			Break
 		}
 		if ($VMHost -eq '') {
 			$VMHost = HOSTNAME.EXE
 		}
 		if (!(Get-VMHost) -icontains $VMHost) {
 			Write-Host -ForegroundColor Red -Object "Host $($VMHost) is not listed in Get-VMHost - terminating backup script."
 			Break
 		}
 		if (!(Get-VM -Server $VMHost)) {
 			Write-Host -ForegroundColor Red -Object "Host $($VMHost) does not appear to have any VMs running on it according to 'Get-VM -Server $($VMHost)'."
 			Write-Host -ForegroundColor Yellow -Object 'This can be occur if PowerShell is not running with elevated privileges.'
 			Write-Host -ForegroundColor Yellow -Object 'Please make sure that you are running PowerShell with Administrator privileges and try again.'
 			Write-Host -ForegroundColor Red -Object 'Terminating backup script.'
 			Break
 		}
 		
 		if ((Test-Path -Path "$($BackupDriveLetter):\VMBackup") -eq $false) {
 			$parentDir = New-Item -Path "$($BackupDriveLetter):\VMBackup" -ItemType 'directory'
 			if ((Test-Path $parentDir) -eq $false) {
 				Write-Host -ForegroundColor Red -Object "Problem creating $parentDir - terminating backup script."
 				Break
 			}
 		}
 		
 		$Logfile = "$($BackupDriveLetter):\VMBackup\Backup-VMs.log"
 		if ((Test-Path $Logfile) -eq $false) {
 			$newFile = New-Item -Path $Logfile -ItemType 'file'
 			if ((Test-Path $Logfile) -eq $false) {
 				Write-Host -ForegroundColor Red -Object "Problem creating $Logfile - terminating backup script."
 				Break
 			}
 		}
 		
 		$backupDate = Get-Date -Format 'yyyy-MM-dd'
 		$destDir = "$($BackupDriveLetter):\VMBackup\$backupDate-$VMHost-backup\"
 		
 		if ((Test-Path $destDir) -eq $false) {
 			$childDir = New-Item -Path $destDir -ItemType 'directory'
 			if ((Test-Path $childDir) -eq $false) {
 				Write-Host -ForegroundColor Red -Object "Problem creating $childDir - terminating backup script."
 				Break
 			}
 		}
 		
 		Add-Content -LiteralPath $Logfile -Value '==================================================================================================='
 		Add-Content -LiteralPath $Logfile -Value '==================================================================================================='
 		Write-InfoDump -text "Starting Hyper-V virtual machine backup for host $VMHost at:"
 		$dateTimeStart = date
 		Write-InfoDump -text "$($dateTimeStart)"
 		Write-InfoDump -text ''
 		
 		Export-MyVms -VMHost $VMHost -Destination $destDir -VMNames $VMNames
 		
 		Write-InfoDump -text ''
 		Write-InfoDump -text 'Exporting finished'
 		
 		
 		$sourceDirectory = Get-ChildItem $destDir
 		
 		if ($sourceDirectory) {
 			$sourceDirSize = Get-ChildItem $destDir -Recurse | Measure-Object -Property length -Sum -ErrorAction SilentlyContinue
 			$sourceDirSize = ($sourceDirSize.sum / 1GB)
 			
 			$hostname = HOSTNAME.EXE
 			$backupDrive = Get-WmiObject -Class win32_logicaldisk -ComputerName $hostname | Where-Object -FilterScript { $_.DeviceID -eq "$($BackupDriveLetter):" }
 			$backupDriveFreeSpace = ($backupDrive.FreeSpace / 1GB)
 			
 			$formattedBackupDriveFreeSpace = '{0:N2}' -f $backupDriveFreeSpace
 			$formattedSourceDirSize = '{0:N2}' -f $sourceDirSize
 			Write-InfoDump -text 'Checking free space for compression:'
 			Write-InfoDump -text "Drive $($BackupDriveLetter): has $formattedBackupDriveFreeSpace GB free on it, this backup took $formattedSourceDirSize GB"
 			
 			$downToOne = $false
 			while (!$downToOne -and $sourceDirSize > $backupDriveFreeSpace) {
 				$backups = Get-ChildItem -Path "$($BackupDriveLetter):\VMBackup\" -Include '*-backup.zip' -Recurse -Name
 				$backups = [array]$backups | Sort-Object
 				
 				if ($backups.length -gt 1) {
 					Write-InfoDump -text "Removing the oldest backup [$($backups[0])] to clear up some more room"
 					Remove-Item -Path "$($BackupDriveLetter):\VMBackup\$($backups[0])" -Recurse -Force
 					$backupDrive = Get-WmiObject -Class win32_logicaldisk -ComputerName $hostname | Where-Object -FilterScript { $_.DeviceID -eq "$($BackupDriveLetter):" }
 					$backupDriveFreeSpace = ($backupDrive.FreeSpace / 1GB)
 					$formattedBackupDriveFreeSpace = '{0:N2}' -f $backupDriveFreeSpace
 					Write-InfoDump -text "Now we have $formattedBackupDriveFreeSpace GB of room"
 				} else {
 					$downToOne = $true
 				}
 			}
 			Write-InfoDump -text 'Compressing the backup...'
 			Invoke-ZipFolder -directory $destDir -VMHost $VMHost
 			
 			$zipFileName = $destDir -replace ".$"
 			$zipFileName = $zipFileName + '.zip'
 			
 			Write-InfoDump -text "Backup [$($zipFileName)] created successfully"
 			$destZipFileSize = (Get-ChildItem $zipFileName).Length / 1GB
 			$formattedDestSize = '{0:N2}' -f $destZipFileSize
 			Write-InfoDump -text "Uncompressed size:`t$formattedSourceDirSize GB"
 			Write-InfoDump -text "Compressed size:  `t$formattedDestSize GB"
 		}
 		
 		Remove-Item $destDir -Recurse -Force
 		
 		Write-InfoDump -text ''
 		Write-InfoDump -text "Finished backup of $VMHost at:"
 		$dateTimeEnd = date
 		Write-InfoDump -text "$($dateTimeEnd)"
 		$length = ($dateTimeEnd - $dateTimeStart).TotalMinutes
 		$length = '{0:N2}' -f $length
 		Write-InfoDump -text "The operation took $length minutes"
 		
 		if ($ShutHostDownWhenFinished -eq $true) {
 			Write-InfoDump -text "Attempting to shut down host machine $VMHost"
 			Invoke-ShutdownTheHost -HostToShutDown $VMHost
 		}
 	}
 }
 
 function Invoke-ShutdownTheHost {
 	<#
 			.SYNOPSIS
 			this function will shut down any vms running on the host executing this script and then shut down said host
 	
 			.DESCRIPTION
 			this function will shut down any vms running on the host executing this script and then shut down said host
 	
 			.PARAMETER HostToShutDown
 			A description of the HostToShutDown parameter.
 	
 			.NOTES
 			My Version of Backup Hyper-V VMs by jhjbhjb h
 	#>
 	
 	[CmdletBinding(SupportsShouldProcess = $true)]
 	param
 	(
 		[System.String]$HostToShutDown
 	)
 	
 	process {
 		$VMs = Get-VM -Server $HostToShutDown
 		
 		if ($VMs) {
 			foreach ($VM in @($VMs)) {
 				$VMName = $VM.ElementName
 				$summofvm = get-vmsummary $VMName
 				$summhb = $summofvm.heartbeat
 				$summes = $summofvm.enabledstate
 				
 				if ($summhb -eq 'OK') {
 					Write-InfoDump -text ''
 					Write-InfoDump -text "HeartBeat Service for $VMName is responding $summhb, saving the machine state"
 					
 					Save-VM -VM $VMName -Server $VMHost -Force -Wait
 				} elseif (($summes -eq 'Stopped') -or ($summes -eq 'Suspended')) {
 					Write-InfoDump -text ''
 					Write-InfoDump -text "$VMName is $summes"
 				} elseif ($summhb -ne 'OK' -and $summes -ne 'Stopped') {
 					Write-InfoDump -text
 					Write-InfoDump -text "HeartBeat Service for $VMName is responding $summhb. Aborting save state."
 				}
 			}
 			Write-InfoDump -text "All VMs on $HostToShutDown shut down or suspended."
 		}
 		Write-InfoDump -text "Shutting down machine $HostToShutDown..."
 		Stop-Computer -ComputerName $HostToShutDown
 	}
 }
 
 
 function Test-IsFileLocked {
 	<#
 			.SYNOPSIS
 			syn
 	
 			.DESCRIPTION
 			desc
 	
 			.PARAMETER path
 			A description of the path parameter.
 	
 			.NOTES
 			My Version of Backup Hyper-V VMs by jhjbhjb h
 	#>
 	
 	[CmdletBinding()]
 	param
 	(
 		[System.String]$path
 	)
 	
 	[bool]$fileExists = Test-Path $path
 	
 	If ($fileExists -eq $false) {
 		Throw 'File does not exist (' + $path + ')'
 	}
 	
 	[bool]$isFileLocked = $true
 	
 	$file = $null
 	
 	Try {
 		$file = [IO.File]::Open(
 			$path,
 			[IO.FileMode]::Open,
 			[IO.FileAccess]::Read,
 		[IO.FileShare]::None)
 		
 		$isFileLocked = $false
 	} Catch [IO.IOException] {
 		If ($_.Exception.Message.EndsWith('it is being used by another process.') -eq $false) {
 			Throw $_.Exception
 		}
 	} Finally {
 		If ($file -ne $null) {
 			$file.Close()
 		}
 	}
 	
 	return $isFileLocked
 }
 
 function Wait-ForZipOperationToFinish {
 	<#
 			.SYNOPSIS
 			syn
 	
 			.DESCRIPTION
 			desc
 	
 			.PARAMETER zipFile
 			A description of the zipFile parameter.
 	
 			.PARAMETER expectedNumberOfItemsInZipFile
 			A description of the expectedNumberOfItemsInZipFile parameter.
 	
 			.NOTES
 			My Version of Backup Hyper-V VMs by jhjbhjb h
 	#>
 	
 	[CmdletBinding()]
 	param
 	(
 		[__ComObject]$zipFile,
 		[System.Int64]$expectedNumberOfItemsInZipFile
 	)
 	
 	Write-InfoDump -text "Waiting for zip operation to finish on $($zipFile.Self.Path)..."
 	
 	[bool]$isFileLocked = Test-IsFileLocked($zipFile.Self.Path)
 	while ($isFileLocked) {
 		Write-Host -NoNewline -Object '.'
 		Start-Sleep -Seconds 5
 		
 		$isFileLocked = Test-IsFileLocked($zipFile.Self.Path)
 	}
 	
 	Write-InfoDump -text ''
 }
 
 function Invoke-ZipFolder {
 	<#
 			.SYNOPSIS
 			syn
 	
 			.DESCRIPTION
 			desc
 	
 			.PARAMETER directory
 			A description of the directory parameter.
 	
 			.NOTES
 			My Version of Backup Hyper-V VMs by jhjbhjb h
 	#>
 	
 	[CmdletBinding()]
 	param
 	(
 		[IO.DirectoryInfo]$directory
 	)
 	
 	$backupFullName = $directory.FullName
 	
 	Write-InfoDump -text ("Creating zip file for folder ($backupFullName)...")
 	
 	[IO.DirectoryInfo]$parentDir = $directory.Parent
 	
 	[string]$zipFileName
 	
 	If ($parentDir.FullName.EndsWith('\') -eq $true) {
 		$zipFileName = $parentDir.FullName + $directory.Name + '.zip'
 	} Else {
 		$zipFileName = $parentDir.FullName + '\' + $directory.Name + '.zip'
 	}
 	
 	Set-Content $zipFileName -Value ('PK' + [char]5 + [char]6 + ("$([char]0)" * 18))
 	
 	$shellApp = New-Object -ComObject Shell.Application
 	$zipFile = $shellApp.NameSpace($zipFileName)
 	
 	If ($zipFile -eq $null) {
 		Write-InfoDump -text 'Failed to get zip file object.'
 	}
 	
 	[int]$expectedCount = (Get-ChildItem $directory -Force -Recurse).Count
 	
 	Write-InfoDump -text "Copying $expectedCount items into file $zipFileName..."
 	
 	$zipFile.CopyHere($directory.FullName)
 	
 	Wait-ForZipOperationToFinish $zipFile $expectedCount
 	
 	Write-InfoDump -text "Successfully created zip file for folder ($backupFullName)."
 }
 
 <#
 		Powershell Script to Shutdown and Export Hyper-V 2008 R2 VMs, one at a time.   
 		Written by Stan Czerno
 		http://www.czerno.com/default.asp?inc=/html/windows/hyperv/cluster/HyperV_Export_VMs.asp
 		I have modified his approach to suit our purposes
 #>
 function Export-MyVms {
 	<#
 			.SYNOPSIS
 			The export Job itself
 	
 			.DESCRIPTION
 			The export Job itself
 	
 			.PARAMETER Destination
 			A description of the Destination parameter.
 	
 			.PARAMETER VMNames
 			A description of the VMNames parameter.
 	
 			.PARAMETER VMHost
 			A description of the VMHost parameter.
 	
 			.NOTES
 			My Version of Backup Hyper-V VMs by jhjbhjb h
 	#>
 	
 	[CmdletBinding(SupportsShouldProcess = $true)]
 	param
 	(
 		[System.String]$Destination,
 		[System.String[]]$VMNames,
 		[System.String]$VMHost
 	)
 	
 	process {
 		<#
 				The script requires the PowerShell Management Library for Hyper-V for it to work. 
 		
 				The PowerShell Management Library for Hyper-V can be downloaded at http://pshyperv.codeplex.com/
 				Be sure to read the documentation before using:
 				http://pshyperv.codeplex.com/releases/view/62842
 				http://pshyperv.codeplex.com/releases/view/38769
 		
 				This is how I backup the VMs on my Two-Node Hyper-V Cluster. I can afford for my servers to be down while this is done and
 				some of my other resources are clustered so there is minimum down time.
 		
 				I also do System State Backups, Exchange Backups and SQL Backups in addition.
 		
 				This script can be used on a Stand-Alone Hyper-V Server as well.
 		
 				Let me know if you have a better way of doing this as I am not a very good developer and new to Powershell.
 		#>
 		
 		if ($VMNames) {
 			if (($VMNames.Length) -gt 1) {
 				$VMs = Get-VM -Name $VMNames -Server $VMHost
 			} else {
 				$VMNames = [string]$VMNames
 				$VMs = Get-VM -Name "$VMNames" -Server $VMHost
 			}
 		} else {
 			$VMs = Get-VM -Server $VMHost
 		}
 		
 		if ($VMs) {
 			foreach ($VM in @($VMs)) {
 				$listOfVmNames += $VM.ElementName + ', '
 			}
 			$listOfVmNames = $listOfVmNames -replace "..$"
 			Write-InfoDump -text 'Attempting to backup the following VMs:'
 			Write-InfoDump -text "$listOfVmNames"
 			Write-InfoDump -text ''
 			Write-Host -Object 'Do not cancel the export process as it may cause unpredictable VM behavior' -ForegroundColor Yellow
 			
 			foreach ($VM in @($VMs)) {
 				$VMName = $VM.ElementName
 				$summofvm = get-vmsummary $VMName
 				$summhb = $summofvm.heartbeat
 				$summes = $summofvm.enabledstate
 				$restartWhenDone = $false
 				
 				$doexport = 'no'
 				
 				if ($summhb -eq 'OK') {
 					$doexport = 'yes'
 					Write-InfoDump -text ''
 					Write-InfoDump -text "HeartBeat Service for $VMName is responding $summhb, saving the machine state"
 					$restartWhenDone = $true
 					
 					Save-VM -VM $VMName -Server $VMHost -Force -Wait
 				} elseif (($summes -eq 'Stopped') -or ($summes -eq 'Suspended')) {
 					$doexport = 'yes'
 					Write-InfoDump -text ''
 					Write-InfoDump -text "$VMName is $summes, starting export"
 				} elseif ($summhb -ne 'OK' -and $summes -ne 'Stopped') {
 					$doexport = 'no'
 					Write-InfoDump -text
 					Write-InfoDump -text "HeartBeat Service for $VMName is responding $summhb. Save state and export aborted for $VMName"
 				}
 				
 				$i = 1
 				if ($doexport -eq 'yes') {
 					$VMState = get-vmsummary $VMName
 					$VMEnabledState = $VMState.enabledstate
 					
 					if ($VMEnabledState -eq 'Suspended' -or $VMEnabledState -eq 'Stopped') {
 						if ([IO.Directory]::Exists("$($Destination)\$($VMName)")) {
 							[IO.Directory]::Delete("$($Destination)\$($VMName)", $true)
 						}
 						Write-InfoDump -text "Exporting $VMName"
 						
 						export-vm -VM $VMName -Server $VMHost -path $Destination -CopyState -Wait -Force -ErrorAction SilentlyContinue
 						
 						$exportedCount = (Get-ChildItem $Destination -Force -Recurse).Count
 						
 						if ($exportedCount -lt 5) {
 							Write-InfoDump -text "***** Automated export failed for $VMName *****"
 							Write-InfoDump -text '***** Manual export advised *****'
 						}
 						
 						if ($restartWhenDone) {
 							Write-InfoDump -text "Restarting $VMName..."
 							
 							Start-VM $VMName -HeartBeatTimeOut 300 -Wait
 						}
 					} else {
 						Write-InfoDump -text "Could not properly save state on VM $VMName, skipping this one and moving on."
 					}
 				}
 			}
 		} else {
 			Write-InfoDump -text 'No VMs found to back up.'
 		}
 	}
 }
 
 function Write-InfoDump {
 	<#
 			.SYNOPSIS
 			This is just a hand-made wrapper function that mimics the Tee-Object cmdlet with less fuss
 	
 			.DESCRIPTION
 			This is just a hand-made wrapper function that mimics the Tee-Object cmdlet with less fuss
 			Plus, it makes our log file prettier
 	
 			.PARAMETER text
 			Text to dump
 	
 			.NOTES
 			Additional information about the function.
 	#>
 	
 	[CmdletBinding(SupportsShouldProcess = $true)]
 	param
 	(
 		[System.Single]$text
 	)
 	
 	process {
 		Write-Host -Object "$text"
 		$now = date
 		$text = "$now`t: $text"
 		Add-Content -LiteralPath $Logfile -Value $text
 	}
 }
`

