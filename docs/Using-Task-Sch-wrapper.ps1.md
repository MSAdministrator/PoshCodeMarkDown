---
Author: battlechicken
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5805
Published Date: 2015-04-01t18
Archived Date: 2015-04-03t14
---

# using task sch, wrapper - 

## Description

powershell example using task scheduler managed wrapper i drafted up some time ago.  the task this creates is harmless; it just launches notepad.exe.  you can run the code and examine the created task to see what you need to change to create a task that does something useful.

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 #
 #
 add-type -Path "C:\PathTo\TaskScheduler\Microsoft.Win32.TaskScheduler.dll"
 
 $taskService = New-Object Microsoft.Win32.TaskScheduler.TaskService
 $taskDef = $taskService.NewTask()
 $taskDef.RegistrationInfo.Description = "TASK THAT I INSTALL ON THE MACHINE"
 
 
 $timeTrigger = New-Object Microsoft.Win32.TaskScheduler.TimeTrigger
 
 $taskDef.Triggers.Add($timeTrigger)
 
 $taskAction = New-Object Microsoft.Win32.TaskScheduler.ExecAction
 $taskAction.Path = "C:\windows\notepad.exe"
 
 $taskDef.actions.Add($taskAction)
 
 $taskDef.Data = "This is a value.  For DATA"
 
 $taskDef.Principal.LogonType = [Microsoft.Win32.TaskScheduler.TaskLogonType]::InteractiveToken
 
 $taskDef.Settings.DisallowStartIfOnBatteries = $false;
 $taskDef.Settings.Enabled = $true
 $taskDef.Settings.Hidden = $false
 $taskDef.Settings.RunOnlyIfIdle = $false
 $taskDef.Settings.RunOnlyIfIdle = $false
 $taskService.RootFolder.RegisterTaskDefinition("BattleChicken's sweet task", $taskDef)
`

