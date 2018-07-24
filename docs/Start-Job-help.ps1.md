---
Author: letoric
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5729
Published Date: 2015-02-05t23
Archived Date: 2015-02-06t05
---

# start-job help - 

## Description

start-job help

## Comments



## Usage



## TODO



## script

`test-transcribing`

## Code

`#
 #
 $ErrorActionPreference = "Stop"
 $DebugPreference = "Continue"
 Set-Location (Split-Path $MyInvocation.MyCommand.Path)
 if(Test-Path (".\" + $MyInvocation.MyCommand.Name.TrimEnd(".ps1") + "_config.xml")) {
         [xml]$configfile = (Get-Content (".\" + $MyInvocation.MyCommand.Name.TrimEnd(".ps1") + "_config.xml"))
 } else {
         Write-Host "Configuration file is missing, halting script" -ForegroundColor Red
         Start-Sleep -Seconds 10
         Exit
 }
  
 $LogDir = $configfile.Root.Configuration.LogDirectory
 $ScriptName = $MyInvocation.MyCommand.Name
 $datetime = (get-date -uformat "%Y-%m-%d_%H-%M-%S")
 $LogFile = ($LogDir + "\" + $ScriptName.TrimEnd(".ps1") + "_" + $datetime + ".log")
 $Global:Transcript = $LogFile
  
 $myWindowsID=[System.Security.Principal.WindowsIdentity]::GetCurrent()
 $myWindowsPrincipal=new-object System.Security.Principal.WindowsPrincipal($myWindowsID)
 $adminRole=[System.Security.Principal.WindowsBuiltInRole]::Administrator
 if ($myWindowsPrincipal.IsInRole($adminRole)) {
    clear-host
 } else {
    $sleeptimer = 10
    $newProcess = new-object System.Diagnostics.ProcessStartInfo "PowerShell";
    $newProcess.WorkingDirectory = (Split-Path $MyInvocation.MyCommand.Path)
    $newProcess.Arguments = $myInvocation.MyCommand.Definition;
    $newProcess.Verb = "runas";
    [System.Diagnostics.Process]::Start($newProcess);
    exit
 }
  
 function Test-Transcribing {
         $externalHost = $host.gettype().getproperty("ExternalHost",
                 [reflection.bindingflags]"NonPublic,Instance").getvalue($host, @())
  
         try {
             $externalHost.gettype().getproperty("IsTranscribing",
                 [reflection.bindingflags]"NonPublic,Instance").getvalue($externalHost, @())
         }
         catch {
                 write-warning "This host does not support transcription."
         }
 }
  
 function main {
         If(!(Test-Path $LogDir)) { New-Item "$LogDir" -Type Directory -Verbose }
         if(Test-Transcribing -eq "True") {
                 Stop-Transcript
                 Start-Transcript
         } else {
                 Start-Transcript
         }
         $a = Get-ChildItem $LogDir | Where-Object {$_.name -like ($ScriptName.TrimEnd(".ps1") + "*")}
         foreach($x in $a) {
                 $y = ((Get-Date) - $x.CreationTime).Days
                 if ($y -gt $configfile.Root.Configuration.MaxLogAge -and $x.PsISContainer -ne $True) { $x.Delete() }
         }
         $JobList = @()
         $StartTime = (Get-Date)
         $integrations = $configfile.root.integrations.integration
         $integrations | ForEach-Object {
                 $ArgumentList = @($_,$configfile)
                 $InitializationScript = {
                         function FTPMonitor {
                                 Param([Parameter(Mandatory=$true)]$integration)
                                 $ArgumentList = @($integration,$args[1])
                                 $ScriptBlock = {
                                         Write-Host ((get-date -uformat "%Y-%m-%d_%H-%M-%S") + ": Starting function(FTPMonitor) to retrieve packages from SFTP for " + $args[0].FolderName + "`r`n")
                                         $PackIn = ($args[1].root.IntegrationDefaults.DefaultInstancePath + $args[0].FolderName + $args[1].root.IntegrationDefaults.DefaultPackInFolder)
                                         If(Test-Path $PackIn) {
                                                 if($args[0].FTPMonitorConfig -eq $null -or $args[0].FTPMonitorConfig -eq "") {
                                                         $IntegrationConfigFile = ($args[1].root.IntegrationDefaults.DefaultConfigurationPath + $env:COMPUTERNAME + "\" + $args[0].FolderName + "\" + $args[0].FolderName + $args[1].root.IntegrationDefaults.DefaultFTPMonitorConfig)
                                                 } else {
                                                         $IntegrationConfigFile = ($args[1].root.IntegrationDefaults.DefaultConfigurationPath + $env:COMPUTERNAME + "\" + $args[0].FolderName + "\" + $args[0].FTPMonitorConfig)
                                                 }
                                                 $NexusCMD = ($args[1].root.IntegrationDefaults.NexusExecutable + " /c " + $IntegrationConfigFile)
 Write-Host $NexusCMD
                                         } else {
                                                 Write-Host ((get-date -uformat "%Y-%m-%d_%H-%M-%S") + ": " + $PackIn + " was not found, unable to process FTP Monitor function")
                                         }
                                         Write-Host ((get-date -uformat "%Y-%m-%d_%H-%M-%S") + ": Ending function(FTPMonitor) to retrieve packages from SFTP for " + $args[0].FolderName + "`r`n")
                                 }
                                 $job = Start-Job -Name ($args[0].FolderName + "_FTPMonitor") -ScriptBlock $ScriptBlock -ArgumentList $ArgumentList | Wait-Job
                                 $job | Receive-Job -Keep
                         }
                         function SJPPack {
                                 Param([Parameter(Mandatory=$true)]$integration)
                                 $ArgumentList = @($integration,$args[1])
                                 $ScriptBlock = {
                                         Write-Host ((get-date -uformat "%Y-%m-%d_%H-%M-%S") + ": Starting function(SJPPack) to pack content for TMS Job submission for " + $args[0].FolderName + "`r`n")
                                         $PackIn = ($args[1].root.IntegrationDefaults.DefaultInstancePath + $args[0].FolderName + $args[1].root.IntegrationDefaults.DefaultPackInFolder)
                                         $PackOut = ($args[1].root.IntegrationDefaults.DefaultInstancePath + $args[0].FolderName + $args[1].root.IntegrationDefaults.DefaultPackOutFolder)
                                         $SJPCMD = ($args[1].root.IntegrationDefaults.SJPExecutable + " /p /m -" + $integration.FolderName + $integration.Environment)
                                         If((Test-Path $PackIn) -and (Test-Path $PackOut)) {
                                                 if((Get-ChildItem $PackIn | Measure-Object).count -gt 1) {
                                                         Write-Host ((get-date -uformat "%Y-%m-%d_%H-%M-%S") + ": Converting integration data into TMS packages")
 Write-Host $SJPCMD
                                                 } else {
                                                         Write-Host ((get-date -uformat "%Y-%m-%d_%H-%M-%S") + ": No data ready to be packaged, moving on")
                                                 }
                                         } else {
                                                 Write-Host ((get-date -uformat "%Y-%m-%d_%H-%M-%S") + ": " + $PackIn + " was not found, unable to process SJP Pack function")
                                         }
                                         Write-Host ((get-date -uformat "%Y-%m-%d_%H-%M-%S") + ": Ending function(SJPPack) to pack content for TMS Job submission for " + $args[0].FolderName + "`r`n")
                                 }
                                 $job = Start-Job -Name ($args[0].FolderName + "_SJP_Pack") -ScriptBlock $ScriptBlock -ArgumentList $ArgumentList | Wait-Job
                                 $job | Receive-Job -Keep
                         }
                         function PKGMonitor {
                                 Param([Parameter(Mandatory=$true)]$integration)
                                 $ArgumentList = @($integration,$args[1])
                                 $ScriptBlock = {
                                         Write-Host ((get-date -uformat "%Y-%m-%d_%H-%M-%S") + ": Starting function(PKGMonitor) to submit jobs to TMS for " + $args[0].FolderName + "`r`n")
                                         $PackOut = ($args[1].root.IntegrationDefaults.DefaultInstancePath + $args[0].FolderName + $args[1].root.IntegrationDefaults.DefaultPackOutFolder)
                                         If(Test-Path $PackOut) {
                                                 if((Get-ChildItem $PackOut | Measure-Object).count -gt 1) {
                                                         Write-Host ((get-date -uformat "%Y-%m-%d_%H-%M-%S") + ": Submitting packages to TMS for processing")
                                                         if($args[0].PKGMonitorConfig -eq $null -or $args[0].PKGMonitorConfig -eq "") {
                                                                 $IntegrationConfigFile = ($args[1].root.IntegrationDefaults.DefaultConfigurationPath + $env:COMPUTERNAME + "\" + $args[0].FolderName + "\" + $args[0].FolderName + $args[1].root.IntegrationDefaults.DefaultPKGMonitorConfig)
                                                         } else {
                                                                 $IntegrationConfigFile = ($args[1].root.IntegrationDefaults.DefaultConfigurationPath + $env:COMPUTERNAME + "\" + $args[0].FolderName + "\" + $args[0].PKGMonitorConfig)
                                                         }
                                                         $NexusCMD = ($args[1].root.IntegrationDefaults.NexusExecutable + " /c " + $IntegrationConfigFile)
 Write-Host $NexusCMD
                                                 } else {
                                                         Write-Host ((get-date -uformat "%Y-%m-%d_%H-%M-%S") + ": No data ready to be submitted to TMS, moving on")
                                                 }
                                         } else {
                                                 Write-Host ((get-date -uformat "%Y-%m-%d_%H-%M-%S") + ": " + $PackOut + " was not found, unable to process PKG Monitor function")
                                         }
                                         Write-Host ((get-date -uformat "%Y-%m-%d_%H-%M-%S") + ": Ending function(PKGMonitor) to submit jobs to TMS for " + $args[0].FolderName + "`r`n")
                                 }
                                 $job = Start-Job -Name ($args[0].FolderName + "_PKGMonitor") -ScriptBlock $ScriptBlock -ArgumentList $ArgumentList | Wait-Job
                                 $job | Receive-Job -Keep
                         }
                         function PKGRetrieve {
                                 Param([Parameter(Mandatory=$true)]$integration)
                                 $ArgumentList = @($integration,$args[1])
                                 $ScriptBlock = {
                                         Write-Host ((get-date -uformat "%Y-%m-%d_%H-%M-%S") + ": Starting function(PKGRetrieve) to retrieve jobs from TMS for " + $args[0].FolderName + "`r`n")
                                         $UnpackIn = ($args[1].root.IntegrationDefaults.DefaultInstancePath + $args[0].FolderName + $args[1].root.IntegrationDefaults.DefaultUnpackInFolder)
                                         If(Test-Path $UnpackIn) {
                                                 if($args[0].PKGRetrieveConfig -eq $null -or $args[0].PKGRetrieveConfig -eq "") {
                                                         $IntegrationConfigFile = ($args[1].root.IntegrationDefaults.DefaultConfigurationPath + $env:COMPUTERNAME + "\" + $args[0].FolderName + "\" + $args[0].FolderName + $args[1].root.IntegrationDefaults.DefaultPKGRetrieveConfig)
                                                 } else {
                                                         $IntegrationConfigFile = ($args[1].root.IntegrationDefaults.DefaultConfigurationPath + $env:COMPUTERNAME + "\" + $args[0].FolderName + "\" + $args[0].PKGRetrieveConfig)
                                                 }
                                                 $NexusCMD = ($args[1].root.IntegrationDefaults.NexusExecutable + " /c " + $IntegrationConfigFile)
 Write-Host $NexusCMD
                                         } else {
                                                 Write-Host ((get-date -uformat "%Y-%m-%d_%H-%M-%S") + ": " + $UnpackIn + " was not found, unable to process PKG Retrieve function")
                                         }
                                         Write-Host ((get-date -uformat "%Y-%m-%d_%H-%M-%S") + ": Ending function(PKGRetrieve) to retrieve jobs from TMS for " + $args[0].FolderName + "`r`n")
                                 }
                                 $job = Start-Job -Name ($args[0].FolderName + "_PKGRetrieve") -ScriptBlock $ScriptBlock -ArgumentList $ArgumentList | Wait-Job
                                 $job | Receive-Job -Keep
                         }
                         function SJPUnpack {
                                 Param([Parameter(Mandatory=$true)]$integration)
                                 $ArgumentList = @($integration,$args[1])
                                 $ScriptBlock = {
                                         Write-Host ((get-date -uformat "%Y-%m-%d_%H-%M-%S") + ": Starting function(SJPUnpack) to unpack content from TMS for return to " + $args[0].FolderName + "`r`n")
                                         $UnpackIn = ($args[1].root.IntegrationDefaults.DefaultInstancePath + $args[0].FolderName + $args[1].root.IntegrationDefaults.DefaultUnpackInFolder)
                                         $UnpackOut = ($args[1].root.IntegrationDefaults.DefaultInstancePath + $args[0].FolderName + $args[1].root.IntegrationDefaults.DefaultUnpackOutFolder)
                                         $SJPCMD = ($args[1].root.IntegrationDefaults.SJPExecutable + " /u /m -" + $integration.FolderName + $integration.Environment)
                                         If((Test-Path $UnpackIn) -and (Test-Path $UnpackOut)) {
                                                 if((Get-ChildItem $UnpackIn | Measure-Object).count -gt 1) {
                                                         Write-Host ((get-date -uformat "%Y-%m-%d_%H-%M-%S") + ": Converting integration data from TMS packages")
 Write-Host $SJPCMD
                                                 } else {
                                                         Write-Host ((get-date -uformat "%Y-%m-%d_%H-%M-%S") + ": No data ready to be unpacked, moving on")
                                                 }
                                         } else {
                                                 Write-Host ((get-date -uformat "%Y-%m-%d_%H-%M-%S") + ": " + $PackOut + " was not found, unable to process SJP Unpack function")
                                         }
                                         Write-Host ((get-date -uformat "%Y-%m-%d_%H-%M-%S") + ": Ending function(SJPUnpack) to unpack content from TMS for return to " + $args[0].FolderName + "`r`n")
                                 }
                                 $job = Start-Job -Name ($args[0].FolderName + "_SJP_Unpack") -ScriptBlock $ScriptBlock -ArgumentList $ArgumentList | Wait-Job
                                 $job | Receive-Job -Keep
                         }
                         function FTPRetrieve {
                                 Param([Parameter(Mandatory=$true)]$integration)
                                 $ArgumentList = @($integration,$args[1])
                                 $ScriptBlock = {
                                         Write-Host ((get-date -uformat "%Y-%m-%d_%H-%M-%S") + ": Starting function(FTPRetrieve) to return translated content to " + $args[0].FolderName + "`r`n")
                                         $UnpackOut = ($args[1].root.IntegrationDefaults.DefaultInstancePath + $args[0].FolderName + $args[1].root.IntegrationDefaults.DefaultUnpackOutFolder)
                                         If(Test-Path $UnpackOut) {
                                                 if($args[0].FTPMonitorConfig -eq $null -or $args[0].FTPMonitorConfig -eq "") {
                                                         $IntegrationConfigFile = ($args[1].root.IntegrationDefaults.DefaultConfigurationPath + $env:COMPUTERNAME + "\" + $args[0].FolderName + "\" + $args[0].FolderName + $args[1].root.IntegrationDefaults.DefaultFTPRetrieveConfig)
                                                 } else {
                                                         $IntegrationConfigFile = ($args[1].root.IntegrationDefaults.DefaultConfigurationPath + $env:COMPUTERNAME + "\" + $args[0].FolderName + "\" + $args[0].FTPRetrieveConfig)
                                                 }
                                                 $NexusCMD = ($args[1].root.IntegrationDefaults.NexusExecutable + " /c " + $IntegrationConfigFile)
 Write-Host $NexusCMD
                                         } else {
                                                 Write-Host ((get-date -uformat "%Y-%m-%d_%H-%M-%S") + ": " + $UnpackOut + " was not found, unable to process FTP Retrieve function")
                                         }
                                         Write-Host ((get-date -uformat "%Y-%m-%d_%H-%M-%S") + ": Ending function(FTPRetrieve) to return translated content to " + $args[0].FolderName + "`r`n")
                                 }
                                 $job = Start-Job -Name ($args[0].FolderName + "_FTPRetrieve") -ScriptBlock $ScriptBlock -ArgumentList $ArgumentList | Wait-Job
                                 $job | Receive-Job -Keep
                         }
                 }
                 $ScriptBlock = {
 Write-Host ((get-date -uformat "%Y-%m-%d_%H-%M-%S") + ": Calling FTPMonitor")
                         FTPMonitor $args[0]
 Write-Host ((get-date -uformat "%Y-%m-%d_%H-%M-%S") + ": Exited FTPMonitor")
  
 Write-Host ((get-date -uformat "%Y-%m-%d_%H-%M-%S") + ": Calling SJPPack")
                         SJPPack $args[0]
 Write-Host ((get-date -uformat "%Y-%m-%d_%H-%M-%S") + ": Exited SJPPack")
                        
 Write-Host ((get-date -uformat "%Y-%m-%d_%H-%M-%S") + ": Calling PKGMonitor")
                         PKGMonitor $args[0]
 Write-Host ((get-date -uformat "%Y-%m-%d_%H-%M-%S") + ": Exited PKGMonitor")
  
 Write-Host ((get-date -uformat "%Y-%m-%d_%H-%M-%S") + ": Calling PKGRetrieve")
                         PKGRetrieve $args[0]
 Write-Host ((get-date -uformat "%Y-%m-%d_%H-%M-%S") + ": Exited PKGRetrieve")
  
 Write-Host ((get-date -uformat "%Y-%m-%d_%H-%M-%S") + ": Calling SJPUnpack")
                         SJPUnpack $args[0]
 Write-Host ((get-date -uformat "%Y-%m-%d_%H-%M-%S") + ": Exited SJPUnpack")
                        
 Write-Host ((get-date -uformat "%Y-%m-%d_%H-%M-%S") + ": Calling FTPRetrieve")
                         FTPRetrieve $args[0]
 Write-Host ((get-date -uformat "%Y-%m-%d_%H-%M-%S") + ": Exited FTPRetrieve")
                 }
                 $ParentJob = Start-Job -Name ($_.FolderName + "_Parent") -InitializationScript $InitializationScript -ScriptBlock $ScriptBlock -ArgumentList $ArgumentList
                 $JobList += $ParentJob
                 if($ParentJob.ChildJobs -ne $null) { $JobList += ($ParentJob.ChildJobs | Get-Job) }
         }
         [INT]$ScriptTimeOut = 1
         $loop = 0
         while ($loop -eq 0 -and (Get-Date) -lt $StartTime.AddMinutes($ScriptTimeOut)) {
                 $jobstatus = (Get-Job -State Running)
                 if($jobstatus -ne $null -and $jobstatus -ne "") {
                         if($DebugPreference -ne "SilentlyContinue") {
                                 Get-Job -State Running | Format-Table -Property ID,Name,State
                         }
 Write-Debug ("`r`n" + (get-date -uformat "%Y-%m-%d_%H-%M-%S") + ": Sleeping for 5 seconds then checking status of jobs again")
                         Start-Sleep -Seconds 5
                 } else {
                         if($DebugPreference -ne "SilentlyContinue") {
                                 Get-Job | Where-Object {$_.state -ne "Running"} | Format-Table -Property ID,Name,State
                         }
 Write-Debug ("`r`n" + (get-date -uformat "%Y-%m-%d_%H-%M-%S") + "All jobs are in a status other than running. Completing script")
                         $loop = 1
                 }
         }
         if((Get-Date) -gt $StartTime.AddMinutes($ScriptTimeOut)) {
                 Write-Host ("`r`n" + (get-date -uformat "%Y-%m-%d_%H-%M-%S") + "Script has exceeded maximum allowed run time, aborting execution")
                 Get-Job -State Running | Format-Table -Property ID,Name,State
                 Get-Job -State Running | Stop-Job
         }
         $JobList | Sort-Object -Property ID | ForEach-Object {
                 $_ | Receive-Job
         }
                
                
                
         if(Test-Transcribing -eq "True") { Stop-Transcript }
         Write-Host "Script is complete!"
         Start-Sleep -Seconds 2
 }
 Get-Job | Remove-Job
 main
`

