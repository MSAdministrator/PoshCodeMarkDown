---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5872
Published Date: 
Archived Date: 2015-05-29t10
---

#  - 

## Description

work in progress reinstall sccm 2007 client, invoke-parallel portion is broken

## Comments



## Usage



## TODO



## script

`remove-ccmnamespace`

## Code

`#
 #
 ###########################################################################################
 #
 #
 ###########################################################################################
 
 If ($PSVersionTable.PSVersion.Major -gt 3) {Write-Host "You have PowerShell Version 4 or higher - you should be fine to run this script"} 
 elseif ($PSVersionTable.PSVersion.Major -eq 3) {Write-Host "You have PowerShell Version 3 - you should be fine to run this script"}
 elseif ($PSVersionTable.PSVersion.Major -eq 2) {Write-Host "You have PowerShell Version 2 - you may have issues running this script"} 
 elseif ($PSVersionTable.PSVersion.Major -eq 1) {Write-Host "You have PowerShell Version 1 - time to upgrade this script is not going to work for you"} 
 
 Start-sleep -seconds 2
 
 if($PSVersionTable.PSVersion.Major -lt 3){$WorkingDirectory = split-path -parent $MyInvocation.MyCommand.Definition}
 else{$WorkingDirectory = $PSScriptRoot}
 
 $Domain = $env:USERDNSDOMAIN
 $WriteHost = $True
 $inputfile = "$WorkingDirectory\Input\servers.txt"
 $logfolder = "$WorkingDirectory\Output\"
 $Trace32 = "$WorkingDirectory\Trace32"
 $timestamp = Get-Date -format "yyyy_MM_dd_HHmmss"
 $resultfile = $logfolder + "SCCM_Client_Reinstall_" + $Domain + "_" + $item + "_" + $timestamp + ".csv"
 
 $bits = "bits"
 $wuauserv = "wuauserv"
 $appidsvc = "appidsvc"
 $cryptsvc = "cryptsvc"
 $ccmexec = "ccmexec"
 
 Set-Alias psexec "$WorkingDirectory\psexec\psexec.exe"
 
 Write-Host "#-------------------------------------------------------------------------------------------------#"
 Write-Host $(Get-Date -format "dd_MM_yyyy_HH:mm:ss") "Running script in Domain: $Domain"
 Write-Host "#-------------------------------------------------------------------------------------------------#"
 
 If (!(Test-Path -Path "$logfolder" -ErrorAction SilentlyContinue)) { New-Item "$logfolder" -Type Directory -ErrorAction SilentlyContinue | Out-Null }
 
 Function Add_To_Log {
 Process {$_ | Add-Content -Path $resultfile}
 }
 Function Remove-CCMNamespace {
 			if (get-wmiobject -query "Select * FROM __Namespace WHERE Name='ccm'" -Namespace "root" -Computername $computer)
 			{
 			Write-Host $(Get-Date -format "dd_MM_yyyy_HH:mm:ss") -foregroundcolor "red" $Computer "Found ccm Namespace"
 			Write-Host $(Get-Date -format "dd_MM_yyyy_HH:mm:ss") -foregroundcolor "yellow" $Computer "Deleting ccm Namespace"
 			get-wmiobject -query "Select * FROM __Namespace WHERE Name='ccm'" -Namespace "root" -Computername $computer | Remove-WMIobject
 			Start-sleep -seconds 2
 			$Status = Get-wmiobject -query "Select * FROM __Namespace WHERE Name='ccm'" -Namespace "root" -computername $Computer
 			if ($Status) {
 			Write-Host $(Get-Date -format "dd_MM_yyyy_HH:mm:ss") -foregroundcolor red "Failed to delete ccm namespace WMI ISSUE"
 			Write-Output "$Computer, Failed to Delete CCM namespace, error,$(Get-Date -format "dd_MM_yyyy_HH:mm:ss")" | Add_To_Log
 			} else {
 			Write-Host $(Get-Date -format "dd_MM_yyyy_HH:mm:ss") -foregroundcolor green $Computer "Successfully removed ccm namespace"}
 			}
             Else {
 			Write-Host $(Get-Date -format "dd_MM_yyyy_HH:mm:ss") -foregroundcolor green $Computer "ccm namespace does not exist"
                   }
 }
 Function Remove-SMSNamespace {
             if (get-wmiobject -query "Select * FROM __Namespace WHERE Name='sms'" -Namespace "root\cimv2" -Computername $computer) {
 			Write-Host $(Get-Date -format "dd_MM_yyyy_HH:mm:ss") -foregroundcolor "red" $Computer "Found SMS Namespace"
 			Write-Host $(Get-Date -format "dd_MM_yyyy_HH:mm:ss") -foregroundcolor "yellow" $Computer "Deleting SMS Namespace"
 			get-wmiobject -query "Select * FROM __Namespace WHERE Name='sms'" -Namespace "root\cimv2" -Computername $computer | Remove-WMIobject
 			Start-sleep -seconds 2
 			$Status = Get-wmiobject -query "Select * FROM __Namespace WHERE Name='sms'" -Namespace "root\cimv2" -computername $Computer
 			if ($Status) {
 			Write-Host $(Get-Date -format "dd_MM_yyyy_HH:mm:ss") -foregroundcolor "RED" "Failed to delete SMS namespace WMI ISSUES"
 			Write-Output "$Computer, Failed to Delete SMS namespace, error,$(Get-Date -format "dd_MM_yyyy_HH:mm:ss")" | Add_To_Log
 			} else {
 			Write-Host $(Get-Date -format "dd_MM_yyyy_HH:mm:ss") -foregroundcolor "Green" $Computer "Successfully removed SMS namespace"}
 			}
             Else {
 			Write-Host $(Get-Date -format "dd_MM_yyyy_HH:mm:ss") -foregroundcolor "Green" $Computer "SMS namespace does not exist"
                   }
 }
 Function Verify-WMI {
 Write-Host $(Get-Date -format "dd_MM_yyyy_HH:mm:ss") -foregroundcolor "yellow" $Item "Running WMI Consistency Check winmgmt /verifyrepository"
 $computername = "\\$item"
 $Scriptblock = "winmgmt /verifyrepository"
 
 $commandLine = "echo . | powershell -Output XML "
 
 $commandBytes = [System.Text.Encoding]::Unicode.GetBytes($scriptblock)
 $encodedCommand = [Convert]::ToBase64String($commandBytes)
 $commandLine += "-EncodedCommand $encodedCommand"
 
 $errorOutput = [IO.Path]::GetTempFileName()
 $output = psexec /acceptEula $computername cmd /c $commandLine 2>$errorOutput
 
 $errorContent = Get-Content $errorOutput
 Remove-Item $errorOutput
 if($errorContent -match "(Access is denied)|(failure)|(Couldn't)")
 {
     $OFS = "`n"
     $errorMessage = "Could not execute remote expression. "
     $errorMessage += "Ensure that your account has administrative " +
         "privileges on the target machine.`n"
     $errorMessage += ($errorContent -match "psexec.exe :")
 
     Write-Error $errorMessage
 }
 
 if ($output -eq "WMI repository is consistent")
 		{
 		Write-Host $(Get-Date -format "dd_MM_yyyy_HH:mm:ss") -foregroundcolor "green" $Item "WMI Check: Success"
 		Write-Output "$Item, WMI Consistency Check, success,$(Get-Date -format "dd_MM_yyyy_HH:mm:ss")" | Add_To_Log
 		return $True
 		}
 		else
 		{
 		Write-Host $(Get-Date -format "dd_MM_yyyy_HH:mm:ss") -foregroundcolor "red" $Item "WMI Check: Failure"
 		Write-Host $(Get-Date -format "dd_MM_yyyy_HH:mm:ss") $Output
 		Write-Output "$Item, WMI Consistency Check, Failed,$(Get-Date -format "dd_MM_yyyy_HH:mm:ss")" | Add_To_Log
 		return $false
 		}
 }
 function Rename-WUSoftwareDistributionFolder {
 					$Path = "\\$Computer\C$\Windows\SoftwareDistribution"
 					$NewName = "SoftwareDistribution" + "_" + $timestamp + ".old"
 					$NewPath = "\\$Computer\C$\Windows\$NewName"
 					
 Write-Host $(Get-Date -format "dd_MM_yyyy_HH:mm:ss") -foregroundcolor yellow $Computer "Attempting to rename SoftwareDistribution Directory"
 Write-Host $(Get-Date -format "dd_MM_yyyy_HH:mm:ss") -foregroundcolor white "from:"
 Write-Host $(Get-Date -format "dd_MM_yyyy_HH:mm:ss") -foregroundcolor cyan "$Path"
 Write-Host $(Get-Date -format "dd_MM_yyyy_HH:mm:ss") -foregroundcolor white "to:"
 Write-Host $(Get-Date -format "dd_MM_yyyy_HH:mm:ss") -foregroundcolor cyan "$NewPath"
 					
 					Rename-item $Path -NewName $NewName
 					
 					Start-sleep -seconds 2
 					
 					if (test-path $NewPath) {
 					Write-Host $(Get-Date -format "dd_MM_yyyy_HH:mm:ss") -foregroundcolor green $Computer "Successfully renamed to: $NewName"
 					}
 					if (!(test-path $path)) {
 							Write-Host $(Get-Date -format "dd_MM_yyyy_HH:mm:ss") -foregroundcolor green $Computer "$Path no longer exists"
 							}					
 						else {					
 							Write-Host $(Get-Date -format "dd_MM_yyyy_HH:mm:ss") -foregroundcolor red $Computer "$Path still exists!"
 						}
 }
 Function Remove-Catroot2Folder {
 $catroot2Path = "\\$computer\C$\windows\syswow64\catroot2"
 $catroot2x86 = "\\$computer\C$\windows\system32\catroot2"
 			
 if (test-path -path $catroot2Path) 
 				{Try
                     {
 					Write-Host $(Get-Date -format "dd_MM_yyyy_HH:mm:ss") -foregroundcolor "yellow" $Computer "Removing $catroot2Path"
 					$files = Get-ChildItem -file -literalpath $catroot2Path -Recurse -force
 					$filecount = $files.count
 					$iteration = 0
 					Foreach ($file in $files) {
 						$iteration++
 						Write-Progress -activity "Removing..." -status "$file ($iteration of $filecount)" -percentcomplete (($iteration/$filecount)*100)
 						Remove-Item $file.fullname -recurse -force
 					}
 					Write-Progress -Completed -activity "Removing..."
 					Write-Host $(Get-Date -format "dd_MM_yyyy_HH:mm:ss") -foregroundcolor cyan "Done with foreach, moving to delete the whole directory and empty folder structure"
 					Remove-item -literalPath $catroot2Path -recurse -force
 					
 					Write-Host $(Get-Date -format "dd_MM_yyyy_HH:mm:ss") -foregroundcolor yellow "checking to see if $catroot2Path exists"
 					Start-sleep -seconds 2
 
 if (test-path -literalpath $catroot2Path) {
 					Write-Host $(Get-Date -format "dd_MM_yyyy_HH:mm:ss") -foregroundcolor red $Computer "Failed to Delete $catroot2Path"
 					Write-Output ("$Computer, Failed to Delete $Catroot2Path, error,$(Get-Date -format "dd_MM_yyyy_HH:mm:ss"),{0}" -f $_.ExceptionMessage) | Add_To_Log
 					} else {Write-Host $(Get-Date -format "dd_MM_yyyy_HH:mm:ss") -foregroundcolor green $Computer "Successfully Deleted $catroot2Path"}
 					} Catch {
 						Write-Output ("$Computer, Failed to Delete $Catroot2Path, error,$(Get-Date -format "dd_MM_yyyy_HH:mm:ss"),{0}" -f $_.ExceptionMessage) | Add_To_Log
                         Write-Host $(Get-Date -format "dd_MM_yyyy_HH:mm:ss") -foregroundcolor "red" $Computer $_.Exception.Message
                             }
                 }
 				
 								
 if (test-path -path $catroot2x86) 
 				{Try
                     {
 					Write-Host $(Get-Date -format "dd_MM_yyyy_HH:mm:ss") -foregroundcolor "yellow" $Computer "Removing $catroot2x86"				
 					$files = Get-ChildItem -file -literalpath $catroot2x86 -Recurse -force
 					$filecount = $files.count
 					$iteration = 0
 					Foreach ($file in $files) {
 						$iteration++
 						Write-Progress -activity "Removing..." -status "$file ($iteration of $filecount)" -percentcomplete (($iteration/$filecount)*100)
 						Remove-Item $file.fullname -recurse -force
 					}
 					Write-Progress -Completed -activity "Removing..."
 					Write-Host $(Get-Date -format "dd_MM_yyyy_HH:mm:ss") -foregroundcolor cyan "Done with foreach, moving to delete the whole directory and empty folder structure"
 					Remove-item -literalPath $catroot2x86 -recurse -force
 					
 					Write-Host $(Get-Date -format "dd_MM_yyyy_HH:mm:ss") -foregroundcolor yellow "checking to see if $catroot2x86 exists"
 					Start-sleep -seconds 2
 					
 					if (test-path -literalpath $catroot2x86) {
 					Write-Host $(Get-Date -format "dd_MM_yyyy_HH:mm:ss") -foregroundcolor red $Computer "Failed to Delete $catroot2x86"
 					Write-Output "$Computer, Failed to Delete $catroot2x86, error,$(Get-Date -format "dd_MM_yyyy_HH:mm:ss")" | Add_To_Log
 					} else {Write-Host $(Get-Date -format "dd_MM_yyyy_HH:mm:ss") -foregroundcolor green $Computer "Successfully Deleted $catroot2x86"}
 					} Catch {
 						Write-Output ("$Computer, Failed to Delete $catroot2x86, error,$(Get-Date -format "dd_MM_yyyy_HH:mm:ss"),{0}" -f $_.ExceptionMessage) | Add_To_Log
                         Write-Host $(Get-Date -format "dd_MM_yyyy_HH:mm:ss") -foregroundcolor "red" ($Computer,$_.Exception.Message)
                             }
                 }
 }
 Function Remove-CCMFolder {
 $ccmPath = "\\$computer\C$\windows\syswow64\ccm"
 $ccmPathx86 = "\\$computer\C$\windows\system32\ccm"
 			
 if (test-path -path $ccmPath) 
 				{Try
                     {
 					Write-Host $(Get-Date -format "dd_MM_yyyy_HH:mm:ss") -foregroundcolor "yellow" $Computer "Removing $ccmPath"
 					$files = Get-ChildItem -file -literalpath $ccmPath -Recurse -force
 					$filecount = $files.count
 					$iteration = 0
 					Foreach ($file in $files) {
 						$iteration++
 						Write-Progress -activity "Removing..." -status "$file ($iteration of $filecount)" -percentcomplete (($iteration/$filecount)*100)
 						Remove-Item $file.fullname -recurse -force
 					}
 					Write-Progress -Completed -activity "Removing..."
 					Write-Host $(Get-Date -format "dd_MM_yyyy_HH:mm:ss") -foregroundcolor cyan "Done with foreach, moving to delete the whole directory and empty folder structure"
 					Remove-item -literalPath $ccmPath -recurse -force
 					
 					Write-Host $(Get-Date -format "dd_MM_yyyy_HH:mm:ss") -foregroundcolor yellow "checking to see if $ccmPath exists"
 					Start-sleep -seconds 2
 
 if (test-path -literalpath $ccmPath) {
 					Write-Host $(Get-Date -format "dd_MM_yyyy_HH:mm:ss") -foregroundcolor red $Computer "Failed to Delete $ccmPath"
 					Write-Output "$Computer, Failed to Delete $ccmPath, error,$(Get-Date -format "dd_MM_yyyy_HH:mm:ss")" | Add_To_Log
 					} else {Write-Host $(Get-Date -format "dd_MM_yyyy_HH:mm:ss") -foregroundcolor green $Computer "Successfully Deleted $ccmPath"}
 					} Catch {
 					Write-Output "$Computer, Failed to Delete $ccmpath, error,$(Get-Date -format "dd_MM_yyyy_HH:mm:ss")" | Add_To_Log
                     Write-Host $(Get-Date -format "dd_MM_yyyy_HH:mm:ss") -foregroundcolor "red" ($Computer,$_.Exception.Message)
                             }
                 }
 				
 								
 if (test-path -path $ccmPathx86) 
 				{Try
                     {
 					Write-Host $(Get-Date -format "dd_MM_yyyy_HH:mm:ss") -foregroundcolor "yellow" $Computer "Removing $ccmPathx86"				
 					$files = Get-ChildItem -file -literalpath $ccmPathx86 -Recurse -force
 					$filecount = $files.count
 					$iteration = 0
 					Foreach ($file in $files) {
 						$iteration++
 						Write-Progress -activity "Removing..." -status "$file ($iteration of $filecount)" -percentcomplete (($iteration/$filecount)*100)
 						Remove-Item $file.fullname -recurse -force
 					}
 					Write-Progress -Completed -activity "Removing..."
 					Write-Host $(Get-Date -format "dd_MM_yyyy_HH:mm:ss") -foregroundcolor cyan "Done with foreach, moving to delete the whole directory and empty folder structure"
 					Remove-item -literalPath $ccmPathx86 -recurse -force
 					
 					Write-Host $(Get-Date -format "dd_MM_yyyy_HH:mm:ss") -foregroundcolor yellow "checking to see if $ccmPathx86 exists"
 					Start-sleep -seconds 2
 					
 					if (test-path -literalpath $ccmPathx86) {
 					Write-Host $(Get-Date -format "dd_MM_yyyy_HH:mm:ss") -foregroundcolor red $Computer "Failed to Delete $ccmPathx86"
 					Write-Output "$Computer, Failed to Delete $ccmPathx86, error,$(Get-Date -format "dd_MM_yyyy_HH:mm:ss")" | Add_To_Log
 					} else {Write-Host $(Get-Date -format "dd_MM_yyyy_HH:mm:ss") -foregroundcolor green $Computer "Successfully Deleted $ccmPathx86"}
 					} Catch {
                         Write-Output "$Computer, Failed to Delete $ccmPathx86, error,$(Get-Date -format "dd_MM_yyyy_HH:mm:ss")" | Add_To_Log
                         Write-Host $(Get-Date -format "dd_MM_yyyy_HH:mm:ss") -foregroundcolor "red" ($Computer,$_.Exception.Message)
                             }
                 }
 }
 Function Remove-CCMCacheFolder {
 $ccmcachePath = "\\$Computer\C$\windows\syswow64\ccm\cache"
 $ccmcachePathx86 = "\\$Computer\c$\Windows\system32\ccm\cache"
 			
 if (test-path -path $ccmcachePath) 
 				{Try
                     {
 					Write-Host $(Get-Date -format "dd_MM_yyyy_HH:mm:ss") -foregroundcolor "yellow" $Computer "Removing $ccmcachePath"
 					$files = Get-ChildItem -file -literalpath $ccmcachePath -Recurse -force
 					$filecount = $files.count
 					$iteration = 0
 					Foreach ($file in $files) {
 						$iteration++
 						Write-Progress -activity "Removing..." -status "$file ($iteration of $filecount)" -percentcomplete (($iteration/$filecount)*100)
 						Remove-Item $file.fullname -recurse -force
 					}
 					Write-Progress -Completed -activity "Removing..."
 					Write-Host $(Get-Date -format "dd_MM_yyyy_HH:mm:ss") -foregroundcolor cyan "Done with foreach, moving to delete the whole directory and empty folder structure"
 					Remove-item -literalPath $ccmcachePath -recurse -force
 					
 					Write-Host $(Get-Date -format "dd_MM_yyyy_HH:mm:ss") -foregroundcolor yellow "checking to see if $ccmcachePath exists"
 					Start-sleep -seconds 2
 
 if (test-path -literalpath $ccmcachePath) {
 					Write-Host $(Get-Date -format "dd_MM_yyyy_HH:mm:ss") -foregroundcolor red $Computer "Failed to Delete $ccmcachePath"
 					Write-Output "$Computer, Failed to Delete $ccmcachepath, error,$(Get-Date -format "dd_MM_yyyy_HH:mm:ss")" | Add_To_Log
 					} else {Write-Host $(Get-Date -format "dd_MM_yyyy_HH:mm:ss") -foregroundcolor green $Computer "Successfully Deleted $ccmcachePath"}
 					} Catch {
 						Write-Output "$Computer, Failed to Delete $ccmcachepath, error,$(Get-Date -format "dd_MM_yyyy_HH:mm:ss")" | Add_To_Log
                         Write-Host $(Get-Date -format "dd_MM_yyyy_HH:mm:ss") -foregroundcolor "red" ($Computer,$_.Exception.Message)
                             }
                 }
 				
 								
 if (test-path -path $ccmcachePathx86) 
 				{Try
                     {
 					Write-Host $(Get-Date -format "dd_MM_yyyy_HH:mm:ss") -foregroundcolor "yellow" $Computer "Removing $ccmcachePathx86"				
 					$files = Get-ChildItem -file -literalpath $ccmcachePathx86 -Recurse -force
 					$filecount = $files.count
 					$iteration = 0
 					Foreach ($file in $files) {
 						$iteration++
 						Write-Progress -activity "Removing..." -status "$file ($iteration of $filecount)" -percentcomplete (($iteration/$filecount)*100)
 						Remove-Item $file.fullname -recurse -force
 					}
 					Write-Progress -Completed -activity "Removing..."					
 					Write-Host $(Get-Date -format "dd_MM_yyyy_HH:mm:ss") -foregroundcolor cyan "Done with foreach, moving to delete the whole directory and empty folder structure"
 					Remove-item -literalPath $ccmcachePathx86 -recurse -force
 					
 					Write-Host $(Get-Date -format "dd_MM_yyyy_HH:mm:ss") -foregroundcolor yellow "checking to see if $ccmcachePathx86 exists"
 					Start-sleep -seconds 2
 					
 					if (test-path -literalpath $ccmcachePathx86) {
 					Write-Host $(Get-Date -format "dd_MM_yyyy_HH:mm:ss") -foregroundcolor red $Computer "Failed to Delete $ccmcachePathx86"
 					Write-Output "$Computer, Failed to Delete $ccmcachepathx86, error,$(Get-Date -format "dd_MM_yyyy_HH:mm:ss")" | Add_To_Log
 					} else {Write-Host $(Get-Date -format "dd_MM_yyyy_HH:mm:ss") -foregroundcolor green $Computer "Successfully Deleted $ccmcachePathx86"}
 					} Catch {
 						Write-Output "$Computer, Failed to Delete $ccmcachepathx86, error,$(Get-Date -format "dd_MM_yyyy_HH:mm:ss")" | Add_To_Log
                         Write-Host $(Get-Date -format "dd_MM_yyyy_HH:mm:ss") -foregroundcolor "red" ($Computer,$_.Exception.Message)
                             }
                 }
 }
 Function Remove-CCMSetupFolder {
 $ccmsetupPath = "\\$Computer\C$\windows\ccmsetup"
 $ccmsetupPathx86 = "\\$computer\C$\windows\system32\ccmsetup"
 			
 if (test-path -path $ccmsetupPath) 
 				{Try
                     {
 					Write-Host $(Get-Date -format "dd_MM_yyyy_HH:mm:ss") -foregroundcolor "yellow" $Computer "Removing $ccmsetupPath"
 					$files = Get-ChildItem -file -literalpath $ccmsetupPath -Recurse -force
 					$filecount = $files.count
 					$iteration = 0
 					Foreach ($file in $files) {
 						$iteration++
 						Write-Progress -activity "Removing..." -status "$file ($iteration of $filecount)" -percentcomplete (($iteration/$filecount)*100)
 						Remove-Item $file.fullname -recurse -force
 					}
 					Write-Progress -Completed -activity "Removing..."
 					Write-Host $(Get-Date -format "dd_MM_yyyy_HH:mm:ss") -foregroundcolor cyan "Done with foreach, moving to delete the whole directory and empty folder structure"
 					Remove-item -literalPath $ccmsetupPath -recurse -force
 					
 					Write-Host $(Get-Date -format "dd_MM_yyyy_HH:mm:ss") -foregroundcolor yellow "checking to see if $ccmsetupPath exists"
 					Start-sleep -seconds 2
 
 if (test-path -literalpath $ccmsetupPath) {
 					Write-Host $(Get-Date -format "dd_MM_yyyy_HH:mm:ss") -foregroundcolor red $Computer "Failed to Delete $ccmsetupPath"
 					Write-Output "$Computer, Failed to Delete $ccmsetupPath, error,$(Get-Date -format "dd_MM_yyyy_HH:mm:ss")" | Add_To_Log
 					} else {Write-Host $(Get-Date -format "dd_MM_yyyy_HH:mm:ss") -foregroundcolor green $Computer "Successfully Deleted $ccmsetupPath"}
 					} Catch {
 						Write-Output "$Computer, Failed to Delete $ccmsetupPath, error,$(Get-Date -format "dd_MM_yyyy_HH:mm:ss")" | Add_To_Log
                         Write-Host $(Get-Date -format "dd_MM_yyyy_HH:mm:ss") -foregroundcolor "red" ($Computer,$_.Exception.Message)
                             }
                 }
 				
 								
 if (test-path -path $ccmsetupPathx86) 
 				{Try
                     {
 					Write-Host $(Get-Date -format "dd_MM_yyyy_HH:mm:ss") -foregroundcolor "yellow" $Computer "Removing $ccmsetupPathx86"				
 					$files = Get-ChildItem -file -literalpath $ccmsetupPathx86 -Recurse -force
 					$filecount = $files.count
 					$iteration = 0
 					Foreach ($file in $files) {
 						$iteration++
 						Write-Progress -activity "Removing..." -status "$file ($iteration of $filecount)" -percentcomplete (($iteration/$filecount)*100)
 						Remove-Item $file.fullname -recurse -force
 					}
 					Write-Progress -Completed -activity "Removing..."
 					Write-Host $(Get-Date -format "dd_MM_yyyy_HH:mm:ss") -foregroundcolor cyan "Done with foreach, moving to delete the whole directory and empty folder structure"
 					Remove-item -literalPath $ccmsetupPathx86 -recurse -force
 					
 					Write-Host $(Get-Date -format "dd_MM_yyyy_HH:mm:ss") -foregroundcolor yellow "checking to see if $ccmsetupPathx86 exists"
 					Start-sleep -seconds 2
 					
 					if (test-path -literalpath $ccmsetupPathx86) {
 					Write-Output "$Computer, Found $ccmsetupPathx86, error,$(Get-Date -format "dd_MM_yyyy_HH:mm:ss")" | Add_To_Log
 					} else {Write-Host $(Get-Date -format "dd_MM_yyyy_HH:mm:ss") -foregroundcolor green $Computer "Successfully Deleted $ccmsetupPathx86"}
 					} Catch {
 						Write-Output "$Computer, Found $ccmsetupPathx86, error,$(Get-Date -format "dd_MM_yyyy_HH:mm:ss")" | Add_To_Log
                         Write-Host $(Get-Date -format "dd_MM_yyyy_HH:mm:ss") -foregroundcolor "red" ($Computer,$_.Exception.Message)
                             }
                 }
 }
 Function Remove-qmgrdatfiles {
 $qmgrdatxpPath = "\\$Computer\c$\Documents and Settings\All Users\Application Data\Microsoft\Network\Downloader"
 $qmgrdatwin7Path = "\\$Computer\C$\ProgramData\Microsoft\Network\Downloader" 
 			
 if (test-path -path $qmgrdatwin7Path) 
 				{Try
                     {
 					Write-Host $(Get-Date -format "dd_MM_yyyy_HH:mm:ss") -foregroundcolor "yellow" $Computer "Removing qmgr*.dat files in $qmgrdatwin7Path"
 					$files = Get-ChildItem -file -literalpath $qmgrdatwin7Path -Recurse -force | where {$_.Name -like "qmgr*.dat"}
 					$filecount = $files.count
 					$iteration = 0
 					Foreach ($file in $files) {
 						$iteration++
 						Write-Progress -activity "Removing..." -status "$file ($iteration of $filecount)" -percentcomplete (($iteration/$filecount)*100)
 						Remove-Item $file.fullname -recurse -force
 					}
 					Write-Progress -Completed -activity "Removing..."
 					
 					Write-Host $(Get-Date -format "dd_MM_yyyy_HH:mm:ss") -foregroundcolor yellow "checking to see if $files exist in $qmgrdatwin7Path"
 					Start-sleep -seconds 2
 if (Get-Childitem -literalpath $qmgrdatwin7Path -file | where {$_.Name -like "qmgr*.dat"}) {
 					Write-Host $(Get-Date -format "dd_MM_yyyy_HH:mm:ss") -foregroundcolor red $Computer "Failed to Delete qmgr*.dat files"
 					Write-Output "$Computer, Failed to Delete qmgr*.dat files, error,$(Get-Date -format "dd_MM_yyyy_HH:mm:ss")" | Add_To_Log
 					}
 					else {Write-Host $(Get-Date -format "dd_MM_yyyy_HH:mm:ss") -foregroundcolor green $Computer "Successfully Deleted qmgr*.dat files"}
 					}
 					Catch {
 						Write-Output "$Computer, Failed to Delete $qmgrdatwin7Path, error,$(Get-Date -format "dd_MM_yyyy_HH:mm:ss")" | Add_To_Log
                         Write-Host $(Get-Date -format "dd_MM_yyyy_HH:mm:ss") -foregroundcolor "red" ($Computer,$_.Exception.Message)
                             }
                 }
 if (test-path -path $qmgrdatxpPath) 
 				{Try
                     {
 					Write-Host $(Get-Date -format "dd_MM_yyyy_HH:mm:ss") -foregroundcolor "yellow" $Computer "Removing qmgr*.dat files in $qmgrdatxpPath"
 					$files = Get-ChildItem -file -literalpath $qmgrdatxpPath -Recurse -force | where {$_.Name -like "qmgr*.dat"}
 					$filecount = $files.count
 					$iteration = 0
 					Foreach ($file in $files) {
 						$iteration++
 						Write-Progress -activity "Removing..." -status "$file ($iteration of $filecount)" -percentcomplete (($iteration/$filecount)*100)
 						Remove-Item $file.fullname -recurse -force
 					}
 					Write-Progress -Completed -activity "Removing..."
 					
 					Write-Host $(Get-Date -format "dd_MM_yyyy_HH:mm:ss") -foregroundcolor yellow "checking to see if $files exist in $qmgrdatxpPath"
 					Start-sleep -seconds 2
 
 if (Get-Childitem -literalpath $qmgrdatxpPath -file | where {$_.Name -like "qmgr*.dat"}) {
 					Write-Host $(Get-Date -format "dd_MM_yyyy_HH:mm:ss") -foregroundcolor red $Computer "Failed to Delete qmgr*.dat files"
 					Write-Output "$Computer, Failed to Delete qmgr*.dat files, error,$(Get-Date -format "dd_MM_yyyy_HH:mm:ss")" | Add_To_Log
 					} else {Write-Host $(Get-Date -format "dd_MM_yyyy_HH:mm:ss") -foregroundcolor green $Computer "Successfully Deleted qmgr*.dat files"}
 					} Catch {
 						Write-Output "$Computer, Failed to Delete qmgr*.dat files, error,$(Get-Date -format "dd_MM_yyyy_HH:mm:ss")" | Add_To_Log
                         Write-Host $(Get-Date -format "dd_MM_yyyy_HH:mm:ss") -foregroundcolor "red" ($Computer,$_.Exception.Message)
                             }
                 }
 }
 Function Force-WUAgentDetectnow {
 
 $computername = "\\$computer"
 $Scriptblock = "wuauclt /resetauthorization /detectnow"
 
 $commandLine = "echo . | powershell -Output XML "
 
 $commandBytes = [System.Text.Encoding]::Unicode.GetBytes($scriptblock)
 $encodedCommand = [Convert]::ToBase64String($commandBytes)
 $commandLine += "-EncodedCommand $encodedCommand"
 
 $errorOutput = [IO.Path]::GetTempFileName()
 $output = psexec /acceptEula $computername cmd /c $commandLine 2>$errorOutput
 
 $errorContent = Get-Content $errorOutput
 Remove-Item $errorOutput
 if($errorContent -match "(Access is denied)|(failure)|(Couldn't)")
 {
     $OFS = "`n"
     $errorMessage = "Could not execute remote expression. "
     $errorMessage += "Ensure that your account has administrative " +
         "privileges on the target machine.`n"
     $errorMessage += ($errorContent -match "psexec.exe :")
 
     Write-Error $errorMessage
 }
 
 
 if ($output -eq $null) {
 Write-Host $(Get-Date -format "dd_MM_yyyy_HH:mm:ss") -foregroundcolor "green" $computer "Successfully sent wuauclt /detectnow /resetauthorization"
 } else {
 Write-Host $(Get-Date -format "dd_MM_yyyy_HH:mm:ss") -foregroundcolor "red" $Computer "Something went wrong sending wuauclt /detectnow /resetauthorization"
 Write-Output "$Computer, Something went wrong wuauclt detectnow not sent or received properly.. not so much an issue.. WUAgent should recover by itself in time, error,$(Get-Date -format "dd_MM_yyyy_HH:mm:ss")" | Add_To_Log
 }
 }
 Function Copy-SCCMClientInstallation {
 $destinationparentdir = "\\$Computer\C$\temp"
 $sourcePath = "$WorkingDirectory\Client"
 $destinationPath = "\\$Computer\C$\Temp\SCCMClientInstall\Client"
 
 if (!(Test-path $destinationparentdir)) {
 				Write-Host $(Get-Date -format "dd_MM_yyyy_HH:mm:ss") "Creating Directory" -foregroundcolor "green" $destinationparentdir
 				New-Item "$destinationparentdir" -Type Directory -ErrorAction SilentlyContinue | Out-Null }
 
 if (!(Test-path $destinationPath)) {
 $files = Get-ChildItem -Path $sourcePath -Recurse
 $filecount = $files.count
 $iteration = 0
 
 Write-Host $(Get-Date -format "dd_MM_yyyy_HH:mm:ss") -Foregroundcolor "Yellow" $Computer "Copying from $SourcePath to $DestinationPath"
 
 Foreach ($file in $files) {
     $iteration++
     Write-Progress -activity "Copying..." -status "$file ($iteration of $filecount)" -percentcomplete (($iteration/$filecount)*100)
   
     if ($file.psiscontainer) {$sourcefilecontainer = $file.parent} else {$sourcefilecontainer = $file.directory}
 
     $relativepath = $sourcefilecontainer.fullname.SubString($sourcepath.length)
 
     Copy-Item $file.fullname ($destinationPath + $relativepath)
 }
 Write-Progress -Completed -activity "Copying..."
 }
 	else {
 	Write-Host $(Get-Date -format "dd_MM_yyyy_HH:mm:ss") -Foregroundcolor "yellow" $Computer "SCCM Client installation files already exist on this machine, suggests script has run before"
 	}
 }
 Function LoadCCMsetupLog {
 if (test-path -path \\$Computer\C$\windows\ccmsetup) {Start $Trace32\Trace32.exe \\$Computer\C$\Windows\ccmsetup\ccmsetup.log}
 				elseif (test-path -path \\$Computer\C$\windows\system32\ccmsetup) {Start $Trace32\Trace32.exe \\$Computer\C$\windows\system32\ccmsetup\ccmsetup.log}
 				elseif (test-path -path \\$Computer\C$\winnt) {Start $Trace32\Trace32.exe \\$Computer\C$\winnt\system32\ccmsetup\ccmsetup.log}
 }
 Function Uninstall-SCCMClient {
 Write-Host $(Get-Date -format "dd_MM_yyyy_HH:mm:ss") -foregroundcolor "Yellow" $Computer "Running SCCM Client Uninstall"
 Write-Host $(Get-Date -format "dd_MM_yyyy_HH:mm:ss") -foregroundcolor "green" $Computer Opening Log
 LoadCCMSetupLog
 				if (test-path -path "\\$Computer\C$\windows\syswow64") {
                 Write-Host $(Get-Date -format "dd_MM_yyyy_HH:mm:ss") -foregroundcolor "green" $Computer "Detected x64 running SCCM Uninstall"
 				
 				psexec \\$computer C:\temp\SCCMClientInstall\Client\ccmsetup.exe /uninstall
 				}
 				elseif (test-path -path "\\$Computer\C$\windows\system32") {
                 Write-Host $(Get-Date -format "dd_MM_yyyy_HH:mm:ss") -foregroundcolor "green" $computer "Detected x86 running SCCM Uninstall"
 				
 				psexec \\$computer C:\temp\SCCMClientInstall\Client\ccmsetup.exe /uninstall
 				}
 				
 				Start-sleep -seconds 7
 				Get-process Trace32 | stop-process		
 }
 Function Install-SCCMClient {
                 Write-Host $(Get-Date -format "dd_MM_yyyy_HH:mm:ss") -foregroundcolor "yellow" $Computer "Beginning SCCM Client Reinstall"
 				
 psexec \\$computer C:\temp\SCCMClientInstall\Client\ccmsetup.exe /mp:melysccm1.oceania.cshare.net SMSSITECODE=O01 SMSSLP=melysccm1.oceania.cshare.net CCMLOGLEVEL=0 CCMLOGMAXHISTORY=3 RESETKEYINFORMATION=TRUE FSP=melysccm1.oceania.cshare.net
 
 				Start-sleep -seconds 2
 				Write-Host $(Get-Date -format "dd_MM_yyyy_HH:mm:ss") -foregroundcolor "green" $Computer Opening Log
 				LoadCCMSetupLog
 				CheckifCCMSetupStillRunning		
 }
 Function CheckifCCMsetupStillrunning {
 Start-sleep -seconds 1
 if (Get-process -ComputerName $Computer ccmsetup) {
                 do {
 				$Process = Get-Process -ComputerName $Computer | where {$_.ProcessName -eq "ccmsetup"}
                 Write-Host $(Get-Date -format "dd_MM_yyyy_HH:mm:ss") -foregroundcolor yellow "ccmsetup.exe still running on $Computer - please be patient"
 				Start-Sleep -Seconds 5
                 } until ($Process -eq $null)
                 Write-Host $(Get-Date -format "dd_MM_yyyy_HH:mm:ss") ("{0}: CCMsetup.exe has exited" -f $Computer)
                 }
 }
 Function Confirm-BitsExists {
                 if (Get-Service -Name $bits -Computername $Computer -ErrorAction SilentlyContinue)
                 {
                     Write-Host $(Get-Date -format "dd_MM_yyyy_HH:mm:ss") -foregroundcolor "Green" $Computer "$bits Exists"
                     return $true
                 }
                 Write-Host $(Get-Date -format "dd_MM_yyyy_HH:mm:ss") -foregroundcolor "red" $Computer "$bits does NOT exist on this computer"
                 Write-Output "$Computer, $bits service does NOT exist, error,$(Get-Date -format "dd_MM_yyyy_HH:mm:ss")" | Add_To_Log
                     return $false
 }
 Function Confirm-WuauservExists {   
                 if (Get-Service -Name $wuauserv -Computername $Computer -ErrorAction SilentlyContinue)
                 {
                     Write-Host $(Get-Date -format "dd_MM_yyyy_HH:mm:ss") -foregroundcolor "Green" $Computer "$wuauserv Exists"
                     return $true
                 }
                     Write-Host $(Get-Date -format "dd_MM_yyyy_HH:mm:ss") -foregroundcolor "red" $Computer "$wuauserv does NOT exist on this computer"
 					Write-Output "$Computer, $wuauserv service does not exist, error,$(Get-Date -format "dd_MM_yyyy_HH:mm:ss")" | Add_To_Log
                     return $false
 }
 Function Confirm-AppIDsvcExists {
                 if (Get-Service -Name $appidsvc -Computername $Computer -ErrorAction SilentlyContinue)
                 {
                     Write-Host $(Get-Date -format "dd_MM_yyyy_HH:mm:ss") -foregroundcolor "Green" $Computer "$appidsvc Exists"
                     return $true
                 }
                     Write-Host $(Get-Date -format "dd_MM_yyyy_HH:mm:ss") -foregroundcolor "red" $Computer "$appidsvc does NOT exist on this computer"
 					Write-Output "$Computer, $appidsvc does NOT exist, error,$(Get-Date -format "dd_MM_yyyy_HH:mm:ss")" | Add_To_Log
                     return $false
 }
 Function Confirm-cryptsvcExists {
                 if (Get-Service -Name $cryptsvc -Computername $Computer -ErrorAction SilentlyContinue)
                 {
                     Write-Host $(Get-Date -format "dd_MM_yyyy_HH:mm:ss") -foregroundcolor "Green" $Computer "$cryptsvc Exists"
                     return $true
                 }
                     Write-Host $(Get-Date -format "dd_MM_yyyy_HH:mm:ss") -foregroundcolor "red" $Computer "$cryptsvc does NOT exist on this computer"
 					Write-Output "$Computer, $cryptsvc does NOT exist, error,$(Get-Date -format "dd_MM_yyyy_HH:mm:ss")" | Add_To_Log
                     return $false
 }
 Function Confirm-ccmexecExists {
                 if (Get-Service -Name $ccmexec -Computername $Computer -ErrorAction SilentlyContinue)
                 {
                     Write-Host $(Get-Date -format "dd_MM_yyyy_HH:mm:ss") -foregroundcolor "Green" $Computer "$ccmexec Exists"
                     return $true
                 }
                     Write-Host $(Get-Date -format "dd_MM_yyyy_HH:mm:ss") -foregroundcolor "red" $Computer "$ccmexec does NOT exist on this computer"
 					Write-Output "$Computer, $ccmexec service does NOT exist, error,$(Get-Date -format "dd_MM_yyyy_HH:mm:ss")" | Add_To_Log
                     return $false
 }
 Function Stop-BitsIfItExists {
                 $bitsexists = Confirm-BitsExists $bitsexists
                 if ($bitsexists) {
 										$bitsstatus = Get-Service -computername $Computer -name $bits
 										if ($bitsstatus |  Where {$_.Status -eq "Running"}) 
 											{
 											SC.exe ("\\" + $Computer) CONFIG $bits start= disabled
 											Start-sleep -seconds 2
 											SC.exe ("\\" + $Computer) Stop $bits
 											Write-Host $(Get-Date -format "dd_MM_yyyy_HH:mm:ss") -foregroundcolor "yellow" $Computer "Stopping $bits service"
 											start-sleep 20
 											$bitsstatus2 = Get-Service -computername $Computer -name $bits
 												if ($bitsstatus2 |  Where {$_.Status -eq "Stopped"} ) {
 												Write-Host $(Get-Date -format "dd_MM_yyyy_HH:mm:ss") -foregroundcolor "Green" $Computer "Service successfully stopped...$bits"
 												} else {
 												Write-Host $(Get-Date -format "dd_MM_yyyy_HH:mm:ss") -foregroundcolor "red" $Computer "Failed to stop...$bits"
 												Write-Output "$Computer, $bits Failed to Stop, error,$(Get-Date -format "dd_MM_yyyy_HH:mm:ss")" | Add_To_Log
 												}
 											}
 											elseif ($bitsstatus |  Where {$_.Status -eq "Stopped"}) {
 											Write-Host $(Get-Date -format "dd_MM_yyyy_HH:mm:ss") -foregroundcolor yellow $Computer "$bits already stopped"
 											}
 									}
 }
 Function Stop-WuauservIfItExists {
                 $wuauservexists = Confirm-WuauservExists $wuauservexists
                 if ($wuauservexists) {
 										$wuauservstatus = Get-Service -computername $Computer -name $wuauserv
 										if ($wuauservstatus |  Where {$_.Status -eq "Running"}) 
 											{
 											SC.exe ("\\" + $Computer) CONFIG $wuauserv start= disabled
 											Start-sleep -seconds 2
 											SC.exe ("\\" + $Computer) Stop $wuauserv
 											Write-Host $(Get-Date -format "dd_MM_yyyy_HH:mm:ss") -foregroundcolor "yellow" $Computer "Stopping $wuauserv service"
 											start-sleep 20
 											$wuauservstatus2 = Get-Service -computername $Computer -name $wuauserv
 											if ($wuauservstatus2 |  Where {$_.Status -eq "Stopped"} ) {
 												Write-Host $(Get-Date -format "dd_MM_yyyy_HH:mm:ss") -foregroundcolor "Green" $Computer "Service successfully stopped...$wuauserv"
 												} else {
 												Write-Host $(Get-Date -format "dd_MM_yyyy_HH:mm:ss") -foregroundcolor "red" $Computer "Failed to stop...$wuauserv"
 												Write-Output "$Computer, $wuauserv Failed to Stop, error,$(Get-Date -format "dd_MM_yyyy_HH:mm:ss")" | Add_To_Log
 												}
 											} 
 											elseif ($wuauservstatus |  Where {$_.Status -eq "Stopped"}) {
 											Write-Host $(Get-Date -format "dd_MM_yyyy_HH:mm:ss") -foregroundcolor yellow $Computer "$wuauserv already stopped"
 											}
 									}
 }
 Function Stop-appidsvcIfItExists {
                 $appidsvcexists = Confirm-appidsvcExists $appidsvcexists
                 if ($appidsvcexists) {
 										$appidsvcstatus = Get-Service -computername $Computer -name $appidsvc
 										if ($appidsvcstatus |  Where {$_.Status -eq "Running"}) 
 											{
 											SC.exe ("\\" + $Computer) CONFIG $appidsvc start= disabled
 											Start-sleep -seconds 2
 											SC.exe ("\\" + $Computer) Stop $appidsvc
 											Write-Host $(Get-Date -format "dd_MM_yyyy_HH:mm:ss") -foregroundcolor "yellow" $Computer "Stopping $appidsvc service"
 											start-sleep 20
 											$appidsvcstatus2 = Get-Service -computername $Computer -name $appidsvc
 												if ($appidsvcstatus2 |  Where {$_.Status -eq "Stopped"} ) {
 												Write-Host $(Get-Date -format "dd_MM_yyyy_HH:mm:ss") -foregroundcolor "Green" $Computer "Service successfully stopped...$appidsvc"
 												} else {
 												Write-Host $(Get-Date -format "dd_MM_yyyy_HH:mm:ss") -foregroundcolor "red" $Computer "Failed to stop...$appidsvc"
 												Write-Output ("{0}, $appidsvc Failed to stop" -f $Computer) | Add_To_Log
 												Write-Output "$Computer, $appidsvc Failed to Stop, error,$(Get-Date -format "dd_MM_yyyy_HH:mm:ss")" | Add_To_Log
 												}
 											} 
 											elseif ($appidsvcstatus |  Where {$_.Status -eq "Stopped"}) {
 											Write-Host $(Get-Date -format "dd_MM_yyyy_HH:mm:ss") -foregroundcolor yellow $Computer "$appidsvc already stopped"
 											}
 									}
 }
 Function Stop-cryptsvcIfItExists {
                 $cryptsvcexists = Confirm-cryptsvcExists $cryptsvcexists
                 if ($cryptsvcexists) {
 										$cryptsvcstatus = Get-Service -computername $Computer -name $cryptsvc
 										if ($cryptsvcstatus |  Where {$_.Status -eq "Running"}) 
 											{
 											SC.exe ("\\" + $Computer) CONFIG $cryptsvc start= disabled
 											Start-sleep -seconds 2
 											SC.exe ("\\" + $Computer) Stop $cryptsvc
 											Write-Host $(Get-Date -format "dd_MM_yyyy_HH:mm:ss") -foregroundcolor "yellow" $Computer "Stopping $cryptsvc service"
 											start-sleep 20
 											$cryptsvcstatus2 = Get-Service -computername $Computer -name $cryptsvc
 											if ($cryptsvcstatus2 |  Where {$_.Status -eq "Stopped"} ) {
 												Write-Host $(Get-Date -format "dd_MM_yyyy_HH:mm:ss") -foregroundcolor "Green" $Computer "Service successfully stopped...$cryptsvc"
 												} else {
 												Write-Host $(Get-Date -format "dd_MM_yyyy_HH:mm:ss") -foregroundcolor "red" $Computer "Failed to stop...$cryptsvc"
 												Write-Output "$Computer, $cryptsvc Failed to Stop, error,$(Get-Date -format "dd_MM_yyyy_HH:mm:ss")" | Add_To_Log
 												}
 											} 
 											elseif ($cryptsvcstatus |  Where {$_.Status -eq "Stopped"}) {
 											Write-Host $(Get-Date -format "dd_MM_yyyy_HH:mm:ss") -foregroundcolor yellow $Computer "$cryptsvc already stopped"
 											}
 									}
 }
 Function Stop-ccmexecIfItExists {
                 $ccmexecexists = Confirm-ccmexecExists $ccmexecexists
                 if ($ccmexecexists) {
 										$ccmexecstatus = Get-Service -computername $Computer -name $ccmexec
 										if ($ccmexecstatus |  Where {$_.Status -eq "Running"}) 
 											{
 											SC.exe ("\\" + $Computer) Stop $ccmexec
 											Write-Host $(Get-Date -format "dd_MM_yyyy_HH:mm:ss") -foregroundcolor "yellow" $Computer "Stopping $ccmexec service"
 											start-sleep 20
 											$ccmexecstatus2 = Get-Service -computername $Computer -name $ccmexec
 											if ($ccmexecstatus2 |  Where {$_.Status -eq "Stopped"} ) {
 												Write-Host $(Get-Date -format "dd_MM_yyyy_HH:mm:ss") -foregroundcolor "Green" $Computer "Service successfully stopped...$ccmexec"
 												} else {
 												Write-Host $(Get-Date -format "dd_MM_yyyy_HH:mm:ss") -foregroundcolor "red" $Computer "Failed to stop...$ccmexec"
 												Write-Output "$Computer, $ccmexec Failed to stop, error,$(Get-Date -format "dd_MM_yyyy_HH:mm:ss")" | Add_To_Log
 												}
 											} 
 											elseif ($ccmexecstatus |  Where {$_.Status -eq "Stopped"}) {
 											Write-Host $(Get-Date -format "dd_MM_yyyy_HH:mm:ss") -foregroundcolor yellow $Computer "$ccmexec already stopped"
 											}
 									}
 }
 Function Stop-iphlpsvcIfItExists {
                 $iphlpsvcexists = Confirm-iphlpsvcExists $iphlpsvcexists
                 if ($iphlpsvcexists) {
 										$iphlpsvcstatus = Get-Service -computername $Computer -name $iphlpsvc
 										if ($iphlpsvcstatus |  Where {$_.Status -eq "Running"}) 
 											{
 											SC.exe ("\\" + $Computer) CONFIG $iphlpsvc start= disabled
 											Start-sleep -seconds 2
 											SC.exe ("\\" + $Computer) Stop $iphlpsvc
 											Write-Host $(Get-Date -format "dd_MM_yyyy_HH:mm:ss") -foregroundcolor "yellow" $Computer "Stopping $iphlpsvc service"
 											start-sleep 20
 											$iphlpsvcstatus2 = Get-Service -computername $Computer -name $iphlpsvc
 											if ($iphlpsvcstatus2 |  Where {$_.Status -eq "Stopped"} ) {
 												Write-Host $(Get-Date -format "dd_MM_yyyy_HH:mm:ss") -foregroundcolor "Green" $Computer "Service successfully stopped...$iphlpsvc"
 												} else {
 												Write-Host $(Get-Date -format "dd_MM_yyyy_HH:mm:ss") -foregroundcolor "red" $Computer "Failed to stop...$iphlpsvc"
 												Write-Output "$Computer, $iphlpsvc Failed to stop, error,$(Get-Date -format "dd_MM_yyyy_HH:mm:ss")" | Add_To_Log
 												}
 											} 
 											elseif ($iphlpsvcstatus |  Where {$_.Status -eq "Stopped"}) {
 											Write-Host $(Get-Date -format "dd_MM_yyyy_HH:mm:ss") -foregroundcolor yellow $Computer "$iphlpsvc already stopped"
 											}
 									}
 }
 Function Start-WuauservIfItExists {
 $wuauservexists = Confirm-wuauservExists $wuauservexists
 if ($wuauservexists) {
 				SC.exe ("\\" + $Computer) CONFIG $wuauserv start= auto
 				Start-Sleep -seconds 2
 				SC.exe ("\\" + $Computer) Start $Wuauserv
                 Write-Host $(Get-Date -format "dd_MM_yyyy_HH:mm:ss") -foregroundcolor "yellow" $Computer ("Service Starting..." + $Wuauserv)
 				Start-Sleep -Seconds 20
 				$status = Get-Service -computername $Computer $Wuauserv | where {$_.Status -eq "Running"}
 				if (($status).status -eq "Running") 
 				{
 				Write-Host $(Get-Date -format "dd_MM_yyyy_HH:mm:ss") -foregroundcolor "Green" $Computer ("Service Started Successfully..." + $Wuauserv)
 				}
 				else {
 				Write-Host $(Get-Date -format "dd_MM_yyyy_HH:mm:ss") -foregroundcolor "Red" $Computer ("Service failed to start..." + $Wuauserv)
 				Write-Output "$Computer, $Wuauserv Failed to start, error,$(Get-Date -format "dd_MM_yyyy_HH:mm:ss")" | Add_To_Log
 				}
 				}
 }
 Function Start-AppIDsvcIfItExists {
 $AppIDsvcexists = Confirm-AppIDsvcExists $AppIDsvcexists
 if ($AppIDsvcexists) {
 				SC.exe ("\\" + $Computer) CONFIG $appidsvc start= auto
 				Start-Sleep -seconds 2
 				SC.exe ("\\" + $Computer) Start $AppIDsvc
                 Write-Host $(Get-Date -format "dd_MM_yyyy_HH:mm:ss") -foregroundcolor "yellow" $Computer ("Service Starting..." + $AppIDsvc)
 				Start-Sleep -Seconds 20
 				$status = Get-Service -computername $Computer $AppIDsvc | where {$_.Status -eq "Running"}
 				if (($status).status -eq "Running") 
 				{
 				Write-Host $(Get-Date -format "dd_MM_yyyy_HH:mm:ss") -foregroundcolor "Green" $Computer ("Service Started Successfully..." + $AppIDsvc)
 				}
 				else {
 				Write-Host $(Get-Date -format "dd_MM_yyyy_HH:mm:ss") -foregroundcolor "Red" $Computer ("Service failed to start..." + $AppIDsvc)
 				Write-Output "$Computer, $AppIDsvc failed to start, error,$(Get-Date -format "dd_MM_yyyy_HH:mm:ss")" | Add_To_Log
 				}
 				}
 }
 Function Start-iphlpsvcIfItExists {
 $iphlpsvcexists = Confirm-iphlpsvcexists $iphlpsvcexists
 if ($iphlpsvcexists) {
 				SC.exe ("\\" + $Computer) CONFIG $iphlpsvc start= auto
 				Start-Sleep -seconds 2
 				SC.exe ("\\" + $Computer) Start $iphlpsvc
                 Write-Host $(Get-Date -format "dd_MM_yyyy_HH:mm:ss") -foregroundcolor "yellow" $Computer ("Service Starting..." + $iphlpsvc)
 				Start-Sleep -Seconds 20
 				$status = Get-Service -computername $Computer $iphlpsvc | where {$_.Status -eq "Running"}
 				if (($status).status -eq "Running") 
 				{
 				Write-Host $(Get-Date -format "dd_MM_yyyy_HH:mm:ss") -foregroundcolor "Green" $Computer ("Service Started Successfully..." + $iphlpsvc)
 				}
 				else {
 				Write-Host $(Get-Date -format "dd_MM_yyyy_HH:mm:ss") -foregroundcolor "Red" $Computer ("Service failed to start..." + $iphlpsvc)
 				Write-Output "$Computer, $iphlpsvc failed to start, error,$(Get-Date -format "dd_MM_yyyy_HH:mm:ss")" | Add_To_Log
 				}
 				}
 }
 Function Start-CryptSvcIfItExists {
 $CryptSvcexists = Confirm-CryptSvcexists $CryptSvcexists
 if ($CryptSvcexists) {
 				SC.exe ("\\" + $Computer) CONFIG $cryptsvc start= auto
 				Start-Sleep -seconds 2
 				SC.exe ("\\" + $Computer) Start $cryptsvc
                 Write-Host $(Get-Date -format "dd_MM_yyyy_HH:mm:ss") -foregroundcolor "yellow" $Computer ("Service Starting..." + $cryptsvc)
 				Start-Sleep -Seconds 20
 				$status = Get-Service -computername $Computer $cryptsvc | where {$_.Status -eq "Running"}
 				if (($status).status -eq "Running")
 				{
 				Write-Host $(Get-Date -format "dd_MM_yyyy_HH:mm:ss") -foregroundcolor "Green" $Computer ("Service Started Successfully..." + $cryptsvc)
 				}
 				else {
 				Write-Host $(Get-Date -format "dd_MM_yyyy_HH:mm:ss") -foregroundcolor "Red" $Computer ("Service failed to start..." + $cryptsvc)
 				Write-Output "$Computer, $cryptsvc failed to start, error,$(Get-Date -format "dd_MM_yyyy_HH:mm:ss")" | Add_To_Log
 				}
 				}
 }
 Function Start-bitsIfItExists {
 $bitsexists = Confirm-bitsexists $bitsexists
 if ($bitsexists) {
 				Write-Host $(Get-Date -format "dd_MM_yyyy_HH:mm:ss") -foregroundcolor yellow $Computer "Setting $bits to manual startup"
 				SC.exe ("\\" + $Computer) CONFIG $bits start= demand
 				Start-sleep -seconds 2
                 Write-Host $(Get-Date -format "dd_MM_yyyy_HH:mm:ss") -foregroundcolor "green" $Computer "Set $bits service to Manual startup ..it does not need to be started"
 }
 }
 Function Reset-security-descriptor-bits {
 Write-Host $(Get-Date -format "dd_MM_yyyy_HH:mm:ss") -foregroundcolor yellow $Computer "Attempting to reset security descriptor on $bits service"
 $Setbitsdescriptor = SC.exe ("\\" + $Computer) sdset bits "D:(A;;CCLCSWRPWPDTLOCRRC;;;SY)(A;;CCDCLCSWRPWPDTLOCRSDRCWDWO;;;BA)(A;;CCLCSWLOCRRC;;;AU)(A;;CCLCSWRPWPDTLOCRRC;;;PU)"
 If ($Setbitsdescriptor -eq "[SC] SetServiceObjectSecurity SUCCESS") {
 Write-Host $(Get-Date -format "dd_MM_yyyy_HH:mm:ss") -foregroundcolor green $Computer "Successfully Reset Security Descriptor on $bits service"
 }
 else {
 Write-Host $(Get-Date -format "dd_MM_yyyy_HH:mm:ss") -foregroundcolor red $Computer"Failed to reset security descriptor on $bits service"
 }
 }
 Function Reset-security-descriptor-wuauserv {
 Write-Host $(Get-Date -format "dd_MM_yyyy_HH:mm:ss") -foregroundcolor yellow $Computer "Attempting to reset security descriptor on $wuauserv service"
 
 $Setwuauservdescriptor = SC.exe ("\\" + $Computer) sdset wuauserv "D:(A;;CCLCSWRPWPDTLOCRRC;;;SY)(A;;CCDCLCSWRPWPDTLOCRSDRCWDWO;;;BA)(A;;CCLCSWLOCRRC;;;AU)(A;;CCLCSWRPWPDTLOCRRC;;;PU)"
 If ($Setwuauservdescriptor -eq "[SC] SetServiceObjectSecurity SUCCESS") {
 Write-Host $(Get-Date -format "dd_MM_yyyy_HH:mm:ss") -foregroundcolor green $Computer "Successfully Reset Security Descriptor on $wuauserv service"
 }
 else {
 Write-Host $(Get-Date -format "dd_MM_yyyy_HH:mm:ss") -foregroundcolor red $Computer"Failed to reset security descriptor on $wuauserv service"
 }
 }
 Function reset-Bitsadmin {
 
 $computername = "\\$computer"
 $Scriptblock = "bitsadmin.exe /reset /allusers"
 
 $commandLine = "echo . | powershell -Output XML "
 
 $commandBytes = [System.Text.Encoding]::Unicode.GetBytes($scriptblock)
 $encodedCommand = [Convert]::ToBase64String($commandBytes)
 $commandLine += "-EncodedCommand $encodedCommand"
 
 $errorOutput = [IO.Path]::GetTempFileName()
 $output = psexec /acceptEula $computername cmd /c $commandLine 2>$errorOutput
 
 $errorContent = Get-Content $errorOutput
 Remove-Item $errorOutput
 if($errorContent -match "(Access is denied)|(failure)|(Couldn't)")
 {
     $OFS = "`n"
     $errorMessage = "Could not execute remote expression. "
     $errorMessage += "Ensure that your account has administrative " +
         "privileges on the target machine.`n"
     $errorMessage += ($errorContent -match "psexec.exe :")
 
     Write-Error $errorMessage
 }
 
 
 $output
 }
 Function registerWU-atl-dll {
 
 $computername = "\\$computer"
 $Scriptblock = "regsvr32.exe /s atl.dll"
 
 Write-Host $(Get-Date -format "dd_MM_yyyy_HH:mm:ss") -foregroundcolor yellow $Computer "Attempting $Scriptblock"
 
 $commandLine = "echo . | powershell -Output XML "
 
 $commandBytes = [System.Text.Encoding]::Unicode.GetBytes($scriptblock)
 $encodedCommand = [Convert]::ToBase64String($commandBytes)
 $commandLine += "-EncodedCommand $encodedCommand"
 
 $errorOutput = [IO.Path]::GetTempFileName()
 $output = psexec /acceptEula $computername cmd /c $commandLine 2>$errorOutput
 
 $errorContent = Get-Content $errorOutput
 Remove-Item $errorOutput
 if($errorContent -match "(Access is denied)|(failure)|(Couldn't)")
 {
     $OFS = "`n"
     $errorMessage = "Could not execute remote expression. "
     $errorMessage += "Ensure that your account has administrative " +
         "privileges on the target machine.`n"
     $errorMessage += ($errorContent -match "psexec.exe :")
 
     Write-Error $errorMessage
 }
 
 
 $output
 }
 Function registerWU-urlmon-dll {
 
 $computername = "\\$computer"
 $Scriptblock = "regsvr32.exe /s urlmon.dll"
 
 Write-Host $(Get-Date -format "dd_MM_yyyy_HH:mm:ss") -foregroundcolor yellow $Computer "Attempting $Scriptblock"
 
 $commandLine = "echo . | powershell -Output XML "
 
 $commandBytes = [System.Text.Encoding]::Unicode.GetBytes($scriptblock)
 $encodedCommand = [Convert]::ToBase64String($commandBytes)
 $commandLine += "-EncodedCommand $encodedCommand"
 
 $errorOutput = [IO.Path]::GetTempFileName()
 $output = psexec /acceptEula $computername cmd /c $commandLine 2>$errorOutput
 
 $errorContent = Get-Content $errorOutput
 Remove-Item $errorOutput
 if($errorContent -match "(Access is denied)|(failure)|(Couldn't)")
 {
     $OFS = "`n"
     $errorMessage = "Could not execute remote expression. "
     $errorMessage += "Ensure that your account has administrative " +
         "privileges on the target machine.`n"
     $errorMessage += ($errorContent -match "psexec.exe :")
 
     Write-Error $errorMessage
 }
 
 
 $output
 }
 Function registerWU-mshtml-dll {
 
 $computername = "\\$computer"
 $Scriptblock = "regsvr32.exe /s mshtml.dll"
 
 Write-Host $(Get-Date -format "dd_MM_yyyy_HH:mm:ss") -foregroundcolor yellow $Computer "Attempting $Scriptblock"
 
 $commandLine = "echo . | powershell -Output XML "
 
 $commandBytes = [System.Text.Encoding]::Unicode.GetBytes($scriptblock)
 $encodedCommand = [Convert]::ToBase64String($commandBytes)
 $commandLine += "-EncodedCommand $encodedCommand"
 
 $errorOutput = [IO.Path]::GetTempFileName()
 $output = psexec /acceptEula $computername cmd /c $commandLine 2>$errorOutput
 
 $errorContent = Get-Content $errorOutput
 Remove-Item $errorOutput
 if($errorContent -match "(Access is denied)|(failure)|(Couldn't)")
 {
     $OFS = "`n"
     $errorMessage = "Could not execute remote expression. "
     $errorMessage += "Ensure that your account has administrative " +
         "privileges on the target machine.`n"
     $errorMessage += ($errorContent -match "psexec.exe :")
 
     Write-Error $errorMessage
 }
 
 
 $output
 }
 Function registerWU-shdocvw-dll {
 
 $computername = "\\$computer"
 $Scriptblock = "regsvr32.exe /s shdocvw.dll"
 
 Write-Host $(Get-Date -format "dd_MM_yyyy_HH:mm:ss") -foregroundcolor yellow $Computer "Attempting $Scriptblock"
 
 $commandLine = "echo . | powershell -Output XML "
 
 $commandBytes = [System.Text.Encoding]::Unicode.GetBytes($scriptblock)
 $encodedCommand = [Convert]::ToBase64String($commandBytes)
 $commandLine += "-EncodedCommand $encodedCommand"
 
 $errorOutput = [IO.Path]::GetTempFileName()
 $output = psexec /acceptEula $computername cmd /c $commandLine 2>$errorOutput
 
 $errorContent = Get-Content $errorOutput
 Remove-Item $errorOutput
 if($errorContent -match "(Access is denied)|(failure)|(Couldn't)")
 {
     $OFS = "`n"
     $errorMessage = "Could not execute remote expression. "
     $errorMessage += "Ensure that your account has administrative " +
         "privileges on the target machine.`n"
     $errorMessage += ($errorContent -match "psexec.exe :")
 
     Write-Error $errorMessage
 }
 
 
 $output
 }
 Function registerWU-browseui-dll {
 
 $computername = "\\$computer"
 $Scriptblock = "regsvr32.exe /s browseui.dll"
 
 Write-Host $(Get-Date -format "dd_MM_yyyy_HH:mm:ss") -foregroundcolor yellow $Computer "Attempting $Scriptblock"
 
 $commandLine = "echo . | powershell -Output XML "
 
 $commandBytes = [System.Text.Encoding]::Unicode.GetBytes($scriptblock)
 $encodedCommand = [Convert]::ToBase64String($commandBytes)
 $commandLine += "-EncodedCommand $encodedCommand"
 
 $errorOutput = [IO.Path]::GetTempFileName()
 $output = psexec /acceptEula $computername cmd /c $commandLine 2>$errorOutput
 
 $errorContent = Get-Content $errorOutput
 Remove-Item $errorOutput
 if($errorContent -match "(Access is denied)|(failure)|(Couldn't)")
 {
     $OFS = "`n"
     $errorMessage = "Could not execute remote expression. "
     $errorMessage += "Ensure that your account has administrative " +
         "privileges on the target machine.`n"
     $errorMessage += ($errorContent -match "psexec.exe :")
 
     Write-Error $errorMessage
 }
 
 
 $output
 }
 Function registerWU-jscript-dll {
 
 $computername = "\\$computer"
 $Scriptblock = "regsvr32.exe /s jscript.dll"
 
 Write-Host $(Get-Date -format "dd_MM_yyyy_HH:mm:ss") -foregroundcolor yellow $Computer "Attempting $Scriptblock"
 
 $commandLine = "echo . | powershell -Output XML "
 
 $commandBytes = [System.Text.Encoding]::Unicode.GetBytes($scriptblock)
 $encodedCommand = [Convert]::ToBase64String($commandBytes)
 $commandLine += "-EncodedCommand $encodedCommand"
 
 $errorOutput = [IO.Path]::GetTempFileName()
 $output = psexec /acceptEula $computername cmd /c $commandLine 2>$errorOutput
 
 $errorContent = Get-Content $errorOutput
 Remove-Item $errorOutput
 if($errorContent -match "(Access is denied)|(failure)|(Couldn't)")
 {
     $OFS = "`n"
     $errorMessage = "Could not execute remote expression. "
     $errorMessage += "Ensure that your account has administrative " +
         "privileges on the target machine.`n"
     $errorMessage += ($errorContent -match "psexec.exe :")
 
     Write-Error $errorMessage
 }
 
 
 $output
 }
 Function registerWU-vbscript-dll {
 
 $computername = "\\$computer"
 $Scriptblock = "regsvr32.exe /s vbscript.dll"
 
 Write-Host $(Get-Date -format "dd_MM_yyyy_HH:mm:ss") -foregroundcolor yellow $Computer "Attempting $Scriptblock"
 
 $commandLine = "echo . | powershell -Output XML "
 
 $commandBytes = [System.Text.Encoding]::Unicode.GetBytes($scriptblock)
 $encodedCommand = [Convert]::ToBase64String($commandBytes)
 $commandLine += "-EncodedCommand $encodedCommand"
 
 $errorOutput = [IO.Path]::GetTempFileName()
 $output = psexec /acceptEula $computername cmd /c $commandLine 2>$errorOutput
 
 $errorContent = Get-Content $errorOutput
 Remove-Item $errorOutput
 if($errorContent -match "(Access is denied)|(failure)|(Couldn't)")
 {
     $OFS = "`n"
     $errorMessage = "Could not execute remote expression. "
     $errorMessage += "Ensure that your account has administrative " +
         "privileges on the target machine.`n"
     $errorMessage += ($errorContent -match "psexec.exe :")
 
     Write-Error $errorMessage
 }
 
 
 $output
 }
 Function registerWU-scrrun-dll {
 
 $computername = "\\$computer"
 $Scriptblock = "regsvr32.exe /s scrrun.dll"
 
 Write-Host $(Get-Date -format "dd_MM_yyyy_HH:mm:ss") -foregroundcolor yellow $Computer "Attempting $Scriptblock"
 
`

