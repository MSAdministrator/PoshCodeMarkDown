---
Author: lee holmes
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5694
Published Date: 2016-01-16t16
Archived Date: 2016-05-24t22
---

# register-temporaryevent. - 

## Description

from windows powershell cookbook (o’reilly) by lee holmes

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 ##############################################################################
 ##
 ##
 ##
 ##############################################################################
 
 <#
 
 .SYNOPSIS
 
 Registers an event action for an object, and automatically unregisters
 itself afterward.
 
 .EXAMPLE
 
 PS >$timer = New-Object Timers.Timer
 PS >Register-TemporaryEvent $timer Disposed { [Console]::Beep(100,100) }
 PS >$timer.Dispose()
 PS >Get-EventSubscriber
 PS >Get-Job
 
 #>
 
 param(
     $Object,
 
     $Event,
 
     [ScriptBlock] $Action
 )
 
 Set-StrictMode -Version Latest
 
 $actionText = $action.ToString()
 $actionText += @'
 
 $eventSubscriber | Unregister-Event
 $eventSubscriber.Action | Remove-Job
 '@
 
 $eventAction = [ScriptBlock]::Create($actionText)
 $null = Register-ObjectEvent $object $event -Action $eventAction
`

