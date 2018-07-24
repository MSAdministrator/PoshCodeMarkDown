---
Author: archdeacon
Publisher: 
Copyright: 
Email: 
Version: 0.0
Encoding: ascii
License: cc0
PoshCode ID: 4947
Published Date: 2014-03-02t07
Archived Date: 2017-04-30t11
---

# set-watcher.ps1 - 

## Description

this script will trap events such as usb insertion, file changes, registry key

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 ###############################################################################
 ###############################################################################
 Set-Variable MSEupdate -Value 0 -Scope Global -Description `
     "MSE Database Status"
 
 ###############################################################################
 ###############################################################################
 $action1 = {
     If (!(test-Path variable:\MyEvent)) {
         Set-Variable -name MyEvent -scope global
     }                         #$MyEvent.TargetInstance will display properties.
     $e = $Event.SourceEventArgs.NewEvent
     $volume = $e.TargetInstance.VolumeName
     $serial = $e.TargetInstance.VolumeSerialNumber
     $free   = $e.TargetInstance.FreeSpace
     $size   = $e.TargetInstance.Size
     If ($volume -eq "") {
         $volume = "<BLANK>"
     If (($serial -ne "") -and ($volume -ne 'AIRCARD')) {           
     }
 }
 $query1 = "SELECT * FROM __InstanceCreationEvent WITHIN 10 WHERE `
     TargetInstance ISA 'Win32_LogicalDisk' AND TargetInstance.DriveType = 2"
 
 $errorActionPreference = "SilentlyContinue"
 $status = "FAIL"
 While ($status -ne "OK") {
     try {
         Register-WmiEvent -Query $query1 -SourceIdentifier USBdevice -SupportEvent -Action $action1 `
         | Out-Null
         $status = "OK"
         Write-Host "USB Filter (Source: USBdevice) has been loaded successfully."
         Write-Host ""
     }
     catch {
         Write-Warning "WMI Error - USB filter has failed to load. Retrying ..."
     }
 }
 $errorActionPreference = "Continue"
 ###############################################################################
 ###############################################################################
 $action2 = {
     $eventLog = "Internet Explorer"
     $id = 99
     $e = $Event.SourceEventArgs.NewEvent
     $drive  = $e.TargetInstance.Drive
     $file   = $e.TargetInstance.FileName + "." + `
               $e.TargetInstance.Extension
     $path   = $e.TargetInstance.Path
     $folder = $drive + "\Users\Derf\Downloads"
     Switch ($e.TargetInstance.Extension) {
         'exe'   {$id = 20; break}
         'enu'   {$id = 21; break}
         'ps1'   {$id = 22; break}
         'html'  {$id = 23; break}
         'zip'   {$id = 24; break}
         'rar'   {$id = 25; break}
         'part'  {$id = 26; break}
         'msi'   {$id = 29; break}
         Default {$id = 99}                 
     }
     $formatString = "{0:0.0}Kb"
     $size = $formatString -f ($e.TargetInstance.FileSize/1KB)
     $file = $file.Replace(".part","")
     $desc = "File $file has been added to the $folder folder (Filesize = $size)"
         Write-EventLog -Logname $eventLog -Source Download -EntryType Information -EventID $ID -Message $desc
     }
 }
 $query2 = "SELECT * FROM __InstanceCreationEvent WITHIN 30 WHERE TargetInstance `
     ISA 'CIM_DataFile' AND TargetInstance.Path = '\\Users\\Derf\\Downloads\\' AND `
         (TargetInstance.Drive = 'C:')"
 
 Register-WmiEvent -Query $query2 -SourceIdentifier Download -SupportEvent -Action $action2 `
     | Out-Null
 
 ###############################################################################
 ###############################################################################
 $action3 = {
     $file = $Event.SourceEventArgs.Name
     $changeType = $Event.SourceEventArgs.ChangeType
     If (($GLOBAL:MSEupdate -eq 0) -and (Test-Path .\Prompter.exe)) {
         & 'd:\Scripts\prompter.exe' /Notify update /Title Microsoft Security Essentials Update `
           /Msg Update started: File $file is being updated. /Time
         [Console]::Beep(800,500)
     }
 }
 
 $folder = "c:\ProgramData\Microsoft\Microsoft Antimalware\Definition Updates\"
 $filter = "*.vdm"
 $fsw = New-Object -TypeName System.IO.FileSystemWatcher -ArgumentList `
     $folder, $filter
 $fsw.EnableRaisingEvents   = $true
 
 Register-ObjectEvent -InputObject $fsw -EventName "Changed" `
     -SourceIdentifier FileWatcher -SupportEvent -Action $action3 | Out-Null
 
 ###############################################################################
 ###############################################################################
 $action4 = {
     $file = 'd:\scripts\WinPlayer.vbs'
     & "$env:programfiles\ccleaner\ccleaner.exe" /AUTO
     attrib -s -h 'C:\Users\derf'
 }
 $query4 = "SELECT * FROM Win32_PowerManagementEvent WHERE EventType = '18'"
 
 Register-WmiEvent -Query $query4 -SourceIdentifier UnHibernate -SupportEvent -Action $action4 `
     | Out-Null
 
 ###############################################################################
 ###############################################################################
 $action5 = {
     If (!(test-Path variable:\MyEvent)) {
         Set-Variable -name MyEvent -scope Global
     }
     $GLOBAL:MyEvent = $Event
     $e = $Event.SourceEventArgs.NewEvent
     & 'd:\Scripts\prompter.exe' /Notify update /Title Avast Update Service `
        /Msg Starting database update
     Write-Eventlog -logname Antivirus -source avast -eventID 90 -entryType `
         Information -message "Avast5 Automatic Database Update"
 }
 $query5 = "SELECT * FROM __InstanceCreationEvent WITHIN 180 WHERE `
     TargetInstance ISA 'Win32_Process' AND TargetInstance.Name = 'avast.setup'"
 
 
 ###############################################################################
 ###############################################################################
 $action7 = {
     If (Test-Path "d:\Applications\ActiveSyncToggle.exe") {
         & 'd:\Applications\ActiveSyncToggle.exe'
     } 
 }
 $query7 = "SELECT * FROM __InstanceCreationEvent WITHIN 10 WHERE `
     TargetInstance ISA 'Win32_Process' AND TargetInstance.Name = 'wmplayer.exe'"
 
  
 ###############################################################################
 ###############################################################################
 
 $hive = "HKEY_LOCAL_MACHINE"
 $keyPath = "SOFTWARE\\Microsoft\\Windows\\CurrentVersion\\Run"
 
 $action8 = {
     $HKLM = 'The key HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Run' +
        ' has been modified; check if the change is intentional.'
     $logType = 2
     $shell = New-Object -Com Wscript.Shell
     If (!(test-Path variable:\MyEvent)) {
         Set-Variable -name MyEvent -scope Global
     }
     $GLOBAL:MyEvent = $Event
     $shell.Popup($HKLM,10,'PS Automatic Event Monitor',48) | Out-Null  
     $Shell.LogEvent($logType,$HKLM) | Out-Null 
 }
 $query8 = "SELECT * FROM RegistryKeyChangeEvent WHERE Hive = '$hive' AND KeyPath = '$keyPath'"
 
 Register-WmiEvent -Query $query8 -Namespace 'root\default' `
     -SourceIdentifier HKLMRunKey -SupportEvent -Action $action8 | Out-Null
 
 ###############################################################################
 ###############################################################################
 
 $action9 = {
     If (($GLOBAL:MSEupdate -eq 1) -and (Test-Path .\Prompter.exe)) {
         & 'd:\Scripts\prompter.exe' /Notify mse /Title Microsoft Security Essentials Update `
               /Msg Database update completed. /Time
         [Console]::Beep(800,500)
     }
 }
 
 $query9 = "SELECT * FROM __InstanceCreationEvent WHERE TargetInstance ISA `
              'Win32_NTLogEvent' AND TargetInstance.EventCode = '2000' AND `
                 TargetInstance.LogFile = 'System'"
 
 Register-WmiEvent -Query $query9 -SourceIdentifier 'EndUpdate' -SupportEvent -Action $action9 `
     | Out-Null 
 
 ###############################################################################
 ###############################################################################
 $action10 = {
     $GLOBAL:MyNewEvent = $Event
     $command = "ipconfig.exe /all"
     $date    = ("{0:F}" -f [DateTime]::Now).PadRight(36,' ')
     $string  = ('-' * 80)
     $ping    = "ping -n 1 Yahoo.com"
 
 $myString = @"
 
 $date                              LAN Restart:
 $string
 "@
 
     $GLOBAL:PingStatus = $lastExitCode
     If (($lastExitCode -eq 0) -and (Test-Path $file)) {
         $myString | Out-File -Filepath $file -Encoding ASCII -Append
         Invoke-Expression $command | Out-File -Filepath $file -Encoding ASCII -Append
     } 
 }
 
 
 $query10 = "SELECT * FROM __InstanceCreationEvent WHERE TargetInstance ISA `
              'Win32_NTLogEvent' AND TargetInstance.EventCode = '300' AND `
                 TargetInstance.SourceName = 'Internet' GROUP WITHIN 90 HAVING `
                   NumberOfEvents > 0"
 
 Register-WmiEvent -Query $query10 -SourceIdentifier 'OnlineStatus' -SupportEvent -Action $action10 `
     | Out-Null 
  
 #################################################################################
 #################################################################################
 
 Register-EngineEvent -SourceIdentifier PowerShell.Exiting -SupportEvent -Action {
    $error | ForEach-Object { $_.InvocationInfo.historyId} |
       Where-Object {$_ -gt 0} | 
          ForEach-Object {clear-history -id $_ -errorAction SilentlyContinue}
    Get-History | Sort-Object -Property CommandLine -Unique |
       Where-Object {$_.CommandLine -ne 'exit'} |
          Sort-Object -Property EndExecutionTime -Descending |
             Export-Clixml (Join-Path (Split-Path $profile) history.clixml) 
 }
`

