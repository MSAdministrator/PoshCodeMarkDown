---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2500
Published Date: 
Archived Date: 2015-05-06t18
---

#  - 

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
 ###############################################################################
 
 $action = {
     $pattern = '^(.*)(\s+|\\)([-_a-z0-9]+\.(?!(psc1))[a-ehmpstv1]{3,4})\b'
     $script = $msg = 'BLANK'
     $e = $Event.SourceEventArgs.NewEvent
     $process   = $e.TargetInstance.Name
     $processID = $e.TargetInstance.ProcessId
     $cmdline   = $e.TargetInstance.CommandLine
     If ($cmdline -match $pattern) {
         $script = $matches[3]
         $path = $matches[1]
             $script = 'BLANK'
         }
     }
     Switch ($e.__CLASS) {
         '__InstanceCreationEvent' { 
             If ($script -ne 'BLANK') {
                 $msg  = "Script Job: $script ($processID) started."
                 $time = Get-Date -Format "dd/MM/yyyy HH:mm:ss"
                 $tag  = "$time [$script] start. --> $cmdline"
                 $ID   = "2"
             }
         }
         '__InstanceDeletionEvent' {
             If ($script -ne 'BLANK') { 
                 $msg  = "Script Job: $script ($processID) ended."
                 $time = Get-Date -Format "dd/MM/yyyy HH:mm:ss"
                 $tag  = "$time [$script] ended. --> $cmdline"
                 $ID   = "1"
             }
         }
         Default                   {$_ | Out-Null }
     }
     If ($cmdline.StartsWith("cscript") -and $cmdline.Contains("//logo")) {
     }
     If ($msg -ne 'BLANK') {
         $file = Join-Path (get-Location) "Monitor.txt"
         If (Test-Path $file) {
             $tag | Out-File $file -encoding 'Default' -Append
         }
         Write-LogEvent Scripts $ID $msg
     }
 }    
      
 $query = "SELECT * FROM __InstanceOperationEvent WITHIN 2 WHERE `
      TargetInstance ISA 'Win32_Process' AND `
     (TargetInstance.Name = 'cmd.exe' OR `
      TargetInstance.Name = 'wscript.exe' OR `
      TargetInstance.Name = 'Cscript.exe' OR `
      TargetInstance.Name = 'mshta.exe')"
  
 ###############################################################################
 ###############################################################################
 $action1 = {
     $e = $Event.SourceEventArgs.NewEvent
     $drive  = $e.TargetInstance.DeviceID
     $volume = $e.TargetInstance.VolumeName
     $free   = $e.TargetInstance.FreeSpace
     $size   = $e.TargetInstance.Size
     If ($e.TargetInstance.VolumeSerialNumber -ne "") {
         & 'c:\windows\system32\wscript.exe' $file $drive $volume $free $size
     }
 }
 $query1 = "SELECT * FROM __InstanceCreationEvent WITHIN 10 WHERE `
     TargetInstance ISA 'Win32_LogicalDisk' AND TargetInstance.DriveType = 2"
 
 Register-WmiEvent -Query $query1 -SourceIdentifier USBdevice -Action $action1 `
     | Out-Null
 
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
     $folder = $drive + "\Downloads"
     Switch ($e.TargetInstance.Extension) {
         'exe'   {$id = 20; break}
         'enu'   {$id = 21; break}
         'html'  {$id = 23; break}
         'zip'   {$id = 24; break}
         'rar'   {$id = 25; break}
         'msi'   {$id = 29; break}
         Default {$id = 99}
     }
     $formatString = "{0:##.#}Mb"
     $size = $formatString -f ($e.TargetInstance.FileSize/1KB)
     $desc = "File $file has been added to the $folder folder; filesize = $size" 
     Write-LogEvent $eventLog $ID $desc
 }
 $query2 = "SELECT * FROM __InstanceCreationEvent WITHIN 30 WHERE TargetInstance `
     ISA 'CIM_DataFile' AND TargetInstance.Path = '\\Downloads\\' AND `
         (TargetInstance.Drive = 'C:' OR TargetInstance.Drive = 'E:')"
 
 Register-WmiEvent -Query $query2 -SourceIdentifier Download -Action $action2 `
     | Out-Null
 
 ###############################################################################
 ###############################################################################
 $action3 = {
     $file = $Event.SourceEventArgs.Name
     & 'c:\Scripts\prompt.exe' /Notify update /Title Avast Update Service `
       /Msg Updating file: $file
 }
 
 $folder = "c:\Program Files\Alwil Software\Avast5\Setup"
 $filter = "*.vpx"
 $fsw = New-Object -TypeName System.IO.FileSystemWatcher -ArgumentList `
     $folder, $filter
 $fsw.IncludeSubDirectories = $false
 $fsw.EnableRaisingEvents   = $true
 
 ###############################################################################
 ###############################################################################
 $action4 = {
     $file = "c:\scripts\KeylogBackup.vbs"
     & 'c:\windows\system32\wscript.exe' $file $args
     & "$env:programfiles\ccleaner\ccleaner.exe" /AUTO
 }
 $query4 = "SELECT * FROM Win32_PowerManagementEvent WHERE EventType = '18'"
 
 Register-WmiEvent -Query $query4 -SourceIdentifier UnHibernate -Action $action4 `
     | Out-Null
 
 ###############################################################################
 ###############################################################################
 $action5 = {
     & 'c:\Scripts\prompt.exe' /Notify update /Title Avast Update Service `
        /Msg Starting database update
     Write-Eventlog -logname Antivirus -source avast -eventID 90 -entryType `
         Information -message "Avast5 Automatic Database Update"
 }
 $query5 = "SELECT * FROM __InstanceCreationEvent WITHIN 180 WHERE `
     TargetInstance ISA 'Win32_Process' AND TargetInstance.Name = 'avast.setup'"
 
 Register-WmiEvent -Query $query5 -SourceIdentifier Avast -Action $action5 `
     | Out-Null
 
 ###############################################################################
 ###############################################################################
 $action7 = {
     $e = $Event.SourceEventArgs.NewEvent
     If (Test-Path "e:\downloads\ActiveSyncToggle.exe") {
         & 'e:\downloads\ActiveSyncToggle.exe'
     } 
 }
 $query7 = "SELECT * FROM __InstanceCreationEvent WITHIN 10 WHERE `
     TargetInstance ISA 'Win32_Process' AND TargetInstance.Name = 'wmplayer.exe'"
 
 Register-WmiEvent -Query $query7 -SourceIdentifier MediaPlayer -Action $action7 `
     | Out-Null
  
 ###############################################################################
 ###############################################################################
 $hive = "HKEY_LOCAL_MACHINE"
 $keyPath = "SOFTWARE\\Microsoft\\Windows\\CurrentVersion\\Run"
 
 $action8 = {
     $file = 'c:\Scripts\HKLMrun.vbs' 
     & 'c:\windows\system32\wscript.exe' $file 
 }
 $query8 = "SELECT * FROM RegistryKeyChangeEvent WHERE Hive = '$hive' AND KeyPath = '$keyPath'"
 
 Register-WmiEvent -Query $query8 -Namespace 'root\default' `
     -SourceIdentifier HKLMRunKey -Action $action8 | Out-Null
  
 ###############################################################################
 #
 ###############################################################################
`

