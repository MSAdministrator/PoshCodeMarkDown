---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 2.6
Encoding: ascii
License: cc0
PoshCode ID: 907
Published Date: 
Archived Date: 2009-03-07t23
---

# new-elevatedtask - 

## Description

creates a new “on demand only” scheduled task to run an “elevated” application, and a shortcut to launch it on demand, so you can bypass uac prompting on specific apps.

## Comments



## Usage



## TODO



## function

`new-elevatedtask`

## Code

`#
 #
 ####################################################################################################
 function New-ElevatedTask {
 <#
 .SYNOPSIS
 	Creates a new "On Demand Only" scheduled task to run an "Elevated" application, and a shortcut to launch it.
 .DESCRIPTION
 	New-ElevatedTask creates a scheduled task on Vista or Windows7 which runs "On Demand" with full priviledges (Elevated) and then creates a shortcut to launch it on demand.
 	
    It works by exploiting the import from XML feature of schtasks.exe /Create /XML ... and the ability to run tasks on demand via schtasks.exe /run /tn ...
 	
    You may specify the shortcut path as a folder path (which must exist), with a name for the new file (ending in .lnk), or you may specify one of the "SpecialFolder" names like "QuickLaunch" or "CommonDesktop" followed by the name.  The shortcut feature depends on the New-Shortcut function (a separate script).
    
 	
    NOTE: You MUST run this in an elevated PowerShell instance.
 
 .Example
 	New-ElevatedTask C:\Windows\Notepad.exe
 
    Will create a task to run notepad elevated, and creates a shortcut to run it the current folder, named "Notepad.lnk"
 	
 .Example
 	New-ElevatedTask C:\Windows\Notepad.exe -Shortcut QuickLaunch\Editor.lnk -FriendlyName "Run Notepad" -TaskName "Elevated Text Editor"
 
    Will create a task to run notepad elevated, and names it "Elevated Text Editor". Also creates a shortcut on the QuickLaunch bar with the name "Run Notepad.lnk" and the tooltip "Elevated Text Editor"
 
 .NOTE
    Must be run from an elevated PowerShell instance
    Some features depend on New-Shortcut (which is also available on the repository: http://PoshCode.org/search/New-Shortcut)
 #>
 [CmdletBinding()]
 param(
    $TargetPath       = ""
 ,  $Arguments        = ""
 ,  $WorkingDirectory = $(Split-Path -parent (Resolve-Path $TargetPath))
 ,  $FriendlyName     = $(Split-Path $TargetPath -leaf)
 ,  $TaskName         = $( "Elevated $friendlyName" )
 ,  $IconLocation     = $null
 ,  $ShortcutPath     = $null
 ,  $Hotkey           = $Null
 ,  $UserName         = $null
 ,  $password         = $null
 ,
    [System.Management.Automation.PSCredential]
    $credential       = $null
 )
 
 $SchTasks = Join-Path ([Environment]::GetFolderPath("System")) "schtasks.exe"
 
 if(-not (Test-Path $SchTasks) ) {
 	$SchTasks = Read-Host "You need to set the correct location for the SchTasks.exe application on your system"
 }
 
 
 
 if($args -contains "-?" -or $TargetPath.Trim(" ").Length -eq 0 -or (-not (Test-Path $TargetPath) )) {
 	if( -not($args -contains "-?")) { Write-Error "TargetPath must be an existing file for the scheduled task to point at!" }
 	Write-Host $UseMessage
 	return
 }
 
 
 $xml = @"
 <?xml version="1.0" encoding="UTF-16"?>
 <Task version="1.2" xmlns="http://schemas.microsoft.com/windows/2004/02/mit/task">
   <RegistrationInfo>
     <Author>{4}</Author>
     <Description>Run {0} "As Administrator"</Description>
   </RegistrationInfo>
   <Triggers />
   <Principals>
     <Principal id="Author">
       <UserId>{4}</UserId>
       <LogonType>{5}</LogonType>
       <RunLevel>HighestAvailable</RunLevel>
     </Principal>
   </Principals>
   <Settings>
     <MultipleInstancesPolicy>Parallel</MultipleInstancesPolicy>
     <DisallowStartIfOnBatteries>false</DisallowStartIfOnBatteries>
     <StopIfGoingOnBatteries>false</StopIfGoingOnBatteries>
     <AllowHardTerminate>true</AllowHardTerminate>
     <StartWhenAvailable>false</StartWhenAvailable>
     <RunOnlyIfNetworkAvailable>false</RunOnlyIfNetworkAvailable>
     <AllowStartOnDemand>true</AllowStartOnDemand>
     <Enabled>true</Enabled>
     <Hidden>false</Hidden>
     <RunOnlyIfIdle>false</RunOnlyIfIdle>
     <DisallowStartOnRemoteAppSession>false</DisallowStartOnRemoteAppSession>
     <UseUnifiedSchedulingEngine>false</UseUnifiedSchedulingEngine>
     <WakeToRun>false</WakeToRun>
     <ExecutionTimeLimit>P3D</ExecutionTimeLimit>
     <Priority>7</Priority>
   </Settings>
   <Actions Context="Author">
     <Exec>
       <Command>{1}</Command>
       <Arguments>{2}</Arguments>
       <WorkingDirectory>{3}</WorkingDirectory>
     </Exec>
   </Actions>
 </Task>
 "@ 
 
 $xFile = [IO.Path]::GetTempFileName()
 
 if($UserName -ne $null -and $password -ne $null)  {
   $xml -f $friendlyName, $TargetPath, $arguments, $WorkingDirectory, $UserName, "Password" | set-content $xFile
   &$SchTasks /Create /XML $xFile /TN $taskname /RU $UserName /RP $password
 
 } elseif($UserName -ne $null -and $password -eq $null)  {
   $xml -f $friendlyName, $TargetPath, $arguments, $WorkingDirectory, $UserName, "Password" | set-content $xFile
   &$SchTasks /Create /XML $xFile /TN $taskname /RU $UserName /RP
 
 } elseif($credential -ne $null) {
   $xml -f $friendlyName, $TargetPath, $arguments, $WorkingDirectory, $($Credential.UserName), "Password" | set-content $xFile
 
   &$SchTasks /Create /XML $xFile /TN $taskname /RU $credential.UserName /RP $($Credential.GetNetworkCredential().Password)
 
 } else {
     $UserName = ([Security.Principal.WindowsIdentity]::GetCurrent().Name)
 
   if($password -ne $null) {
     $xml -f $friendlyName, $TargetPath, $arguments, $WorkingDirectory, $UserName, "Password" | set-content $xFile
     &$SchTasks /Create /XML $xFile /TN $taskname /RU $UserName /RP $password
   
   } else {
     $xml -f $friendlyName, $TargetPath, $arguments, $WorkingDirectory, $UserName, "InteractiveToken" | set-content $xFile
     &$SchTasks /Create /XML $xFile /TN $taskname
   }
 }
 
 if($IconLocation -eq $null -or $IconLocation.Length -eq 0) {
 	$IconLocation = $TargetPath
 }
 if(([IO.FileInfo]$ShortcutPath).Extension.Length -eq 0 ) {
     $ShortcutPath = "$ShortcutPath\$FriendlyName.lnk"
 	$Description  = $TaskName
 } else {
 	$Description  = $FriendlyName
 }
 if(Get-Command New-Shortcut -ErrorAction SilentlyContinue) {
    New-Shortcut -Target $SchTasks -LinkPath $ShortcutPath -Arg "/Run /TN `"$TaskName`"" -WorkingDirectory $WorkingDirectory -Window "Minimized" -Icon $IconLocation -Hotkey $Hotkey -Description $Description
 }
 }
`

