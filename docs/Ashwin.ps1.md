---
Author: ashwin rayaprolu
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6496
Published Date: 2016-08-31t01
Archived Date: 2016-09-03t23
---

# ashwin - 

## Description

some of commands get outdated and needs to get updated with latest powershell upgrade

## Comments



## Usage



## TODO



## script

`backup-vms`

## Code

`#
 #
 #
  
 $Logfile = ""
  
 Function Backup-VMs
 {
         [CmdletBinding(SupportsShouldProcess=$True)]
         Param(
                 [parameter(Mandatory = $true)]
                
                 [ValidateNotNullOrEmpty()]
         )
         process
         {
                	Write-Host -ForegroundColor Red "Processsing........"
 
                
                 if ($BackupDriveLetter -like "*:")
                 {
                         $BackupDriveLetter = $BackupDriveLetter -replace ".$"
                 }
                 if ((Test-Path "$($BackupDriveLetter):") -eq $false)
                 {
                         Write-Host -ForegroundColor Red "Drive $($BackupDriveLetter): does not exist - terminating backup script."
                         Break
                 }
                 if ($VMHost -eq "")
                 {
                         $VMHost = Hostname
                 }
                 if (!(Get-VMHost) -icontains $VMHost)
                 {
                         Write-Host -ForegroundColor Red "Host $($VMHost) is not listed in Get-VMHost - terminating backup script."
                         Break
                 }
                 if (!(Get-VM -ComputerName $VMHost))
                 {
                         Write-Host -ForegroundColor Red "Host $($VMHost) does not appear to have any VMs running on it according to 'Get-VM -Server $($VMHost)'."
                         Write-Host -ForegroundColor Yellow "This can be occur if PowerShell is not running with elevated privileges."
                         Write-Host -ForegroundColor Yellow "Please make sure that you are running PowerShell with Administrator privileges and try again."
                         Write-Host -ForegroundColor Red "Terminating backup script."
                         Break
                 }
                
                 if ((Test-Path "$($BackupDriveLetter):\VMBackup") -eq $false)
                 {
                         $parentDir = New-Item -Path "$($BackupDriveLetter):\VMBackup" -ItemType "directory"
                         if ((Test-Path $parentDir) -eq $false)
                         {
                                 Write-Host -ForegroundColor Red "Problem creating $parentDir - terminating backup script."
                                 Break
                         }
                 }
                
                 $Logfile = "$($BackupDriveLetter):\VMBackup\Backup-VMs.log"
                 if ((Test-Path $Logfile) -eq $false)
                 {
                         $newFile = New-Item -Path $Logfile -ItemType "file"
                         if ((Test-Path $Logfile) -eq $false)
                         {
                                 Write-Host -ForegroundColor Red "Problem creating $Logfile - terminating backup script."
                                 Break
                         }
                 }
  
                 $backupDate = Get-Date -Format "yyyy-MM-dd"
                 $destDir = "$($BackupDriveLetter):\VMBackup\$backupDate-$VMHost-backup\"
                
                 if ((Test-Path $destDir) -eq $false)
                 {
                         $childDir = New-Item -Path $destDir -ItemType "directory"
                         if ((Test-Path $childDir) -eq $false)
                         {
                                 Write-Host -ForegroundColor Red "Problem creating $childDir - terminating backup script."
                                 Break
                         }
                 }
                
                 Add-content -LiteralPath $Logfile -value "==================================================================================================="
                 Add-content -LiteralPath $Logfile -value "==================================================================================================="
                 T -text "Starting Hyper-V virtual machine backup for host $VMHost at:"
                 $dateTimeStart = date
                 T -text "$($dateTimeStart)"
                 T -text ""
                
                 ExportMyVms -VMHost $VMHost -Destination $destDir -VMNames $VMNames
                
                 T -text ""
                 T -text "Exporting finished"
                
  
                 $sourceDirectory = Get-ChildItem $destDir
                
                 if ($sourceDirectory)
                 {
                         $sourceDirSize = Get-ChildItem $destDir -Recurse | Measure-Object -property length -sum -ErrorAction SilentlyContinue
                         $sourceDirSize = ($sourceDirSize.sum / 1GB)
                        
                         $hostname = Hostname
                         $backupDrive = Get-WmiObject win32_logicaldisk -ComputerName $hostname | Where-Object { $_.DeviceID -eq "$($BackupDriveLetter):" }
                         $backupDriveFreeSpace = ($backupDrive.FreeSpace / 1GB)
                        
                         $formattedBackupDriveFreeSpace = "{0:N2}" -f $backupDriveFreeSpace
                         $formattedSourceDirSize = "{0:N2}" -f $sourceDirSize
                         T -text "Checking free space for compression:"
                         T -text "Drive $($BackupDriveLetter): has $formattedBackupDriveFreeSpace GB free on it, this backup took $formattedSourceDirSize GB"
                        
                         $downToOne = $false
                         while (!$downToOne -and $sourceDirSize > $backupDriveFreeSpace)
                         {
                                 $backups = Get-ChildItem -Path "$($BackupDriveLetter):\VMBackup\" -include "*-backup.zip" -recurse -name
                                 $backups = [array]$backups | Sort-Object
                                
                                 if ($backups.length -gt 1)
                                 {
                                         T -text "Removing the oldest backup [$($backups[0])] to clear up some more room"
                                         Remove-Item "$($BackupDriveLetter):\VMBackup\$($backups[0])" -Recurse -Force
                                         $backupDrive = Get-WmiObject win32_logicaldisk -ComputerName $hostname | Where-Object { $_.DeviceID -eq "$($BackupDriveLetter):" }
                                         $backupDriveFreeSpace = ($backupDrive.FreeSpace / 1GB)
                                         $formattedBackupDriveFreeSpace = "{0:N2}" -f $backupDriveFreeSpace
                                         T -text "Now we have $formattedBackupDriveFreeSpace GB of room"
                                 }
                                 else
                                 {
                                         $downToOne = $true
                                 }
                         }
                         T -text "Compressing the backup..directory $destDir VMHost $VMHost."
                         ZipFolder -directory $destDir -VMHost $VMHost
                        
                         $zipFileName = $destDir -replace ".$"
                         $zipFileName = $zipFileName + ".zip"
                        
                         T -text "Backup [$($zipFileName)] created successfully"
                         $destZipFileSize = (Get-ChildItem $zipFileName).Length / 1GB
                         $formattedDestSize = "{0:N2}" -f $destZipFileSize
                         T -text "Uncompressed size:`t$formattedSourceDirSize GB"
                         T -text "Compressed size:  `t$formattedDestSize GB"
                 }
                                        
                 Remove-Item $destDir -Recurse -Force
                
                 T -text ""
                 T -text "Finished backup of $VMHost at:"
                 $dateTimeEnd = date
                 T -text "$($dateTimeEnd)"
                 $length = ($dateTimeEnd - $dateTimeStart).TotalMinutes
                 $length = "{0:N2}" -f $length
                 T -text "The operation took $length minutes"
                
                 if ($ShutHostDownWhenFinished -eq $true)
                 {		
                         T -text "Attempting to shut down host machine $VMHost"
                 }
         }
 }
  
 Function ShutdownTheHost
 {
         [CmdletBinding(SupportsShouldProcess=$True)]
         Param(
         [string]$HostToShutDown
         )
         process
         {
                 $VMs = Get-VM -Server $HostToShutDown
                 if ($VMs)
                 {
                         foreach ($VM in @($VMs))
                         {
                                 $VMName = $VM.ElementName
                                 $summofvm = get-vmsummary $VMName
                                 $summhb = $summofvm.heartbeat
                                 $summes = $summofvm.enabledstate
                                
                                 if ($summhb -eq "OK")
                                 {
                                         T -text ""
                                         T -text "HeartBeat Service for $VMName is responding $summhb, saving the machine state"
                                        
                                         Save-VM -VM $VMName -Server $VMHost -Force -Wait
                                 }
                                 elseif (($summes -eq "Stopped") -or ($summes -eq "Suspended"))
                                 {
                                         T -text ""
                                         T -text "$VMName is $summes"
                                 }
                                
                                 elseif ($summhb -ne "OK" -and $summes -ne "Stopped")
                                 {
                                         T -text
                                         T -text "HeartBeat Service for $VMName is responding $summhb. Aborting save state."
                                 }
                         }
                         T -text "All VMs on $HostToShutDown shut down or suspended."
                 }
                 T -text "Shutting down machine $HostToShutDown..."
         }
 }
  
  
 function IsFileLocked(
     [string] $path)
 {    
     [bool] $fileExists = Test-Path $path
    
     If ($fileExists -eq $false)
     {
         Throw "File does not exist (" + $path + ")"
     }
    
     [bool] $isFileLocked = $true
  
     $file = $null
    
     Try
     {
         $file = [IO.File]::Open(
             $path,
             [IO.FileMode]::Open,
             [IO.FileAccess]::Read,
             [IO.FileShare]::None)
            
         $isFileLocked = $false
     }
     Catch [IO.IOException]
     {
         If ($_.Exception.Message.EndsWith("it is being used by another process.") -eq $false)
         {
             Throw $_.Exception
         }
     }
     Finally
     {
         If ($file -ne $null)
         {
             $file.Close()
         }
     }
    
     return $isFileLocked
 }
    
 function WaitForZipOperationToFinish(
     [__ComObject] $zipFile,
     [int] $expectedNumberOfItemsInZipFile)
 {
     T -text "Waiting for zip operation to finish on $($zipFile.Self.Path)..."
    
         [bool] $isFileLocked = IsFileLocked($zipFile.Self.Path)
     while($isFileLocked)
     {
         Write-Host -NoNewLine "."
         Start-Sleep -Seconds 5
        
         $isFileLocked = IsFileLocked($zipFile.Self.Path)
     }
    
     T -text ""    
 }
  
 function ZipFolder(
     [IO.DirectoryInfo] $directory)
 {    
         $backupFullName = $directory.FullName
        
     T -text ("Creating zip file for folder ($backupFullName)...")
    
     [IO.DirectoryInfo] $parentDir = $directory.Parent
    
     [string] $zipFileName
    
     If ($parentDir.FullName.EndsWith("\") -eq $true)
     {
         $zipFileName = $parentDir.FullName + $directory.Name + ".zip"
     }
     Else
     {
         $zipFileName = $parentDir.FullName + "\" + $directory.Name + ".zip"
     }
        
     Set-Content $zipFileName ("PK" + [char]5 + [char]6 + ("$([char]0)" * 18))
        
     $shellApp = New-Object -ComObject Shell.Application
     $zipFile = $shellApp.NameSpace($zipFileName)
  
     If ($zipFile -eq $null)
     {
         T -text "Failed to get zip file object."
     }
    
     [int] $expectedCount = (Get-ChildItem $directory -Force -Recurse).Count
    
         T -text "Copying $expectedCount items into file $zipFileName..."
        
     $zipFile.CopyHere($directory.FullName)
  
     WaitForZipOperationToFinish $zipFile $expectedCount
    
     T -text "Successfully created zip file for folder ($backupFullName)."
 }
 
 
 
 
 $lStr_GetSummaryInfoError = "Get Summary info error : {0}"
 
 
 Function Get-VMSummary
     [CmdletBinding(SupportsShouldProcess=$true)]
     param(
            [parameter(ValueFromPipeline = $true)]
            $VM="%" ,
            
            [ValidateNotNullOrEmpty()]
          )         
     Process {
         if ($VM -is [String]) {$VM=(Get-VM -Name $VM -ComputerName $Server) }
         if ($VM.count -gt 1 ) {[Void]$PSBoundParameters.Remove("VM") ;  $VM | ForEach-object {Get-VMSummary -VM $_  @PSBoundParameters}}
         if ($vm.__CLASS -eq 'Msvm_ComputerSystem') { 
             $VSMgtSvc=Get-WmiObject -computerName $Vm.__Server -NameSpace $HyperVNamespace  -Class "MsVM_virtualSystemManagementService" 
             $result=$VSMgtSvc.GetSummaryInformation( @( (Get-VMSettingData $vm ).__Path ) ,  @(0,1,2,3,4,100,101,102,103,104,105,106,107,108))
             if ($Result.ReturnValue -eq 0) {$result.SummaryInformation | foreach-object {
                  New-Object PSObject -Property @{      
                    "Host"             =  $VM.__server                   ;  "VMElementName"    =  $_.elementname
                    "Name"             =  $_.name                        ;  "CreationTime"     =  $_.CreationTime
                    "EnabledState"     =  ([vmstate]($_.EnabledState))   ;  "Notes"            =  $_.Notes
                    "CPUCount"         =  $_.NumberOfProcessors          ;  "CPULoad"          =  $_.ProcessorLoad
                    "CPULoadHistory"   =  $_.ProcessorLoadHistory        ;  "MemoryUsage"      =  $_.MemoryUsage
                    "GuestOS"          =  $_.GuestOperatingSystem        ;  "Snapshots"        =  $_.Snapshots.count
                    "Jobs"             =  $_.AsynchronousTasks           ;  "Uptime"           =  $_.UpTime
                    "UptimeFormatted"  =  $(if ($_.uptime -gt 0) {([datetime]0).addmilliseconds($_.UpTime).tostring("hh:mm:ss")} else {0} )
                    "Heartbeat"        =  $(if ($_.heartBeat) { [HeartBeatICStatus] $_.Heartbeat} else {$null} )
                    "FQDN"             =  ((get-vmkvp -vm $vm).FullyQualifiedDomainName)
         	       "IpAddress"        =  ((Ping-VM $vm).NetworkAddress)}
             }}
             else  {write-Warning ($lStr_GetSummaryInfo  -f [ReturnCode]$result.returnValue )}
         }
     }
 }
 
 
 
  
 Function ExportMyVms
 {
         [CmdletBinding(SupportsShouldProcess=$True)]
         Param(
         [string]$Destination,
                 [string[]]$VMNames,
                 [string]$VMHost
         )
         process
         {              
  
  
  
  
  
  
                 if ($VMNames)
                 {
                         if (($VMNames.Length) -gt 1)
                         {
                                 $VMs = Get-VM -Name $VMNames -ComputerName $VMHost
                         }
                         else
                         {
                                 $VMNames = [string]$VMNames
                                 $VMs = Get-VM -Name "$VMNames" -ComputerName $VMHost
                         }
                 }
                 else
                 {
                         $VMs = Get-VM -ComputerName $VMHost
                 }
                
                 if ($VMs)
                 {
                         foreach ($VM in @($VMs))
                         {
                                 $listOfVmNames += $VM.Name + ", "
                         }
                         $listOfVmNames = $listOfVmNames -replace "..$"
                         T -text "Attempting to backup the following VMs:"
                         T -text "$listOfVmNames"
                         T -text ""
                         Write-Host "Do not cancel the export process as it may cause unpredictable VM behavior" -ForegroundColor Yellow
                        
                         foreach ($VM in @($VMs))
                         {
                                 $VMName = $VM.Name
                                 $summofvm = get-vm $VMName 
                                 $restartWhenDone = $false
                                
                                 $doexport = "yes"
                                
                                
                                 $i = 1
                                 if ($doexport -eq "yes")
                                 {
                                         $VMState = get-vm $VMName
                                         #$VMEnabledState = $VMState.State
                                         $VMExportingStatus = $VMState.Status
                                        
                                         T -text "Exporting $VMName with state :$VMExportingStatus"
 
                                         if($VMExportingStatus -ne "Exporting virtual machine")
                                         {
                                                 if ([IO.Directory]::Exists("$($Destination)\$($VMName)"))
                                                 {
                                                         [IO.Directory]::Delete("$($Destination)\$($VMName)", $True)
                                                 }
                                                 T -text "Exporting $VMName"
                                                
                                                 export-vm $VMName -ComputerName $VMHost -path $Destination 
                                                
                                                 $exportedCount = (Get-ChildItem $Destination -Force -Recurse).Count
                                                
                                                 if ($exportedCount -lt 5)
                                                 {
     	                                           	 T -text "***** Automated export failed for $VMName *****"
 	                                                 T -text "***** Manual export advised *****"
                                                 }
                                                
                                         }
                                         else
                                         {
                                                 T -text "Could not properly save state on VM $VMName, skipping this one and moving on."
                                         }
                                 }
                         }
                 }
                 else
                 {
                         T -text "No VMs found to back up."
                 }
         }
 }
  
 Function T
 {
         [CmdletBinding(SupportsShouldProcess=$True)]
         Param(
         [string]$text
         )
         process
         {
                 Write-Host "$text"
                 $now = date
                 $text = "$now`t: $text"
                 Add-content -LiteralPath $Logfile -value $text
         }
 }
 
 
 Backup-VMs -BackupDriveLetter Y -VMHost localhost  -VMNames DEVELOPERIIS
`

