---
Author: rottedquickly
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6531
Published Date: 2016-09-26t15
Archived Date: 2016-11-12t02
---

# clear sccmcache - 

## Description

description

## Comments



## Usage



## TODO



## script

`foreach-parallel`

## Code

`#
 #
 
 
     (Optional) Force of CM Actions are added as well. 
 
 .Example
 Run Script from PowerShell ISE in Administrator mode.
 #>
 
 function ForEach-Parallel {
     param(
         [Parameter(Mandatory=$true,position=0)]
         [System.Management.Automation.ScriptBlock] $ScriptBlock,
         [Parameter(Mandatory=$true,ValueFromPipeline=$true)]
         [PSObject]$InputObject,
         [Parameter(Mandatory=$false)]
         [int]$MaxThreads=10
     )
     BEGIN {
         $iss = [system.management.automation.runspaces.initialsessionstate]::CreateDefault()
         $pool = [Runspacefactory]::CreateRunspacePool(1, $maxthreads, $iss, $host)
         $pool.open()
         $threads = @()
         $ScriptBlock = $ExecutionContext.InvokeCommand.NewScriptBlock("param(`$_)`r`n" + $Scriptblock.ToString())
     }
     PROCESS {
         $powershell = [powershell]::Create().addscript($scriptblock).addargument($InputObject)
         $powershell.runspacepool=$pool
         $threads+= @{
             instance = $powershell
             handle = $powershell.begininvoke()
         }
     }
     END {
 
         $notdone = $true
         while ($notdone) {
             $notdone = $false
             for ($i=0; $i -lt $threads.count; $i++) {
                 $thread = $threads[$i]
                 if ($thread) {
                     if ($thread.handle.iscompleted) {
                         $thread.instance.endinvoke($thread.handle)
                         $thread.instance.dispose()
                         $threads[$i] = $null
                     }
                     else {
                         $notdone = $true
                     }
                 }
             }
         }
     }
 }
  
 write-host `n
 Read-Host Press Enter to continue 
 Write-Host "Choose hostnames(computernames list) text file" -ForegroundColor yellow
 ######################################################################################
 ######################################################################################
 function Read-OpenFileDialog([string]$WindowTitle, [string]$InitialDirectory, [string]$Filter = "All files (*.*)|*.*", [switch]$AllowMultiSelect)
 {  
     Add-Type -AssemblyName System.Windows.Forms
     $openFileDialog = New-Object System.Windows.Forms.OpenFileDialog
     $openFileDialog.Title = $WindowTitle
     if ($InitialDirectory -eq $Null) { $openFileDialog.InitialDirectory = $InitialDirectory } 
     $openFileDialog.Filter = $Filter
     if ($AllowMultiSelect) { $openFileDialog.MultiSelect = $true }
     $openFileDialog.ShowHelp = $true    
     $openFileDialog.ShowDialog() > $null
     if ($AllowMultiSelect) { return $openFileDialog.Filenames } else { return $openFileDialog.Filename }
 }
 
 $var = Read-OpenFileDialog("Select hostnames file:","c:\")
 
 $Computers = Get-Content $var
 
 Start-Sleep -s 3
 
  
   #$hostlist = Get-Content 'C:\clh\1.txt'
 #$Computers |ForEach-Parallel -MaxThreads 25
    
  $Computers |ForEach-Parallel -MaxThreads 10 {  
 
  if(Test-Connection $_ -quiet){
 
 
 Invoke-Command -computername $_ -ScriptBlock { 
 
 get-service ccmexec,wuauserv,cryptsvc,bits | where {$_.status -eq 'running'} | stop-service -Force -Verbose
 Start-Sleep -Seconds 10
 
 Remove-Item C:\Windows\SoftwareDistribution\* -Recurse -Force -Verbose
 
 
 
 get-service ccmexec,wuauserv,cryptsvc,bits | where {$_.status -eq 'stopped'} | start-service -Verbose -Passthru
 
 Start-Sleep -Seconds 10
 
 $CMObject = New-Object -ComObject "UIResource.UIResourceMgr" -ErrorAction STOP
         $CMCacheObject = $CMObject.GetCacheInfo()
         foreach($CItem in $CMCacheObject.GetCacheElements()){
             $CMCacheObject.DeleteCacheElement($CItem.CacheElementId)
 }
 
 Start-Sleep -Seconds 10
 
 ([wmiclass]'ROOT\ccm:SMS_Client').TriggerSchedule('{00000000-0000-0000-0000-000000000113}')
 
 ([wmiclass]'ROOT\ccm:SMS_Client').TriggerSchedule('{00000000-0000-0000-0000-000000000108}')
 
 Start-sleep -Seconds 30
 
 $UpdateList = [System.Management.ManagementObject[]](Get-WmiObject -ComputerName $_ -Query 'SELECT * FROM CCM_SoftwareUpdate' -Namespace ROOT\ccm\ClientSDK);([wmiclass]"\\$_\ROOT\ccm\ClientSDK:CCM_SoftwareUpdatesManager").InstallUpdates($UpdateList);
 
 
     }
 
 }
 
 }
`

