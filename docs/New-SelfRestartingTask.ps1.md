---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 5.0
Encoding: ascii
License: cc0
PoshCode ID: 1807
Published Date: 
Archived Date: 2010-05-01t15
---

# new-selfrestartingtask - 

## Description

restarts applications using a managementeventwatcher to watch for an instancedeletionevent … use at your own risk ;)

## Comments



## Usage



## TODO



## function

`new-selfrestartingtask`

## Code

`#
 #
 function New-SelfRestartingTask {
 <#
 .Notes
   For production use you should consider investigating more specific matching in the Query clause.
 
   There are many possibilities here: you could watch for instances of a process with specific command-line parameters, or a certain caption, etc. Or, you could match against one process, but fire off a different one when it crashes. 
 
   Have a look at the Win32_Process class to see all the properties available for matching.
 .Link
   Win32_Process Class http://msdn.microsoft.com/en-us/library/aa394372%28VS.85%29.aspx
 .Link
   __InstanceDeletionEvent http://msdn.microsoft.com/en-us/library/aa394652%28VS.85%29.aspx
 .Link
   ScriptCenter article on Events http://www.microsoft.com/technet/scriptcenter/topics/winpsh/events.mspx
 .Synopsis
   Restarts an application when it exits or crashes
 .Description
   Create a WMIEvent handler to watch for the termination of a specific executable and restart it.
 
 .Parameter Application
   The path to an application you want to restart when it crashes
 
 .Parameter Parameters
   Parameters to be passed to the application when we (re)start it
 
 .Parameter Start
   Automatically start the process (even if it's already running)
 
 .Parameter Interval
   Polling interval to check whether the application has exited or not
 
 .Example
   New-SelfRestartingTask notepad -Start
 
  Description
  -----------
  Starts notepad right away and monitors for crash or exit and restarts it.
 
 .Example
  New-SelfRestartingTask C:\Program` Files\Internet` Explorer\IExplore.exe http://HuddledMasses.org
 
  Description
  -----------
  Monitors for the termination of Internet Explorer and restarts it pointed at my blog.
  Note that this does NOT start IE, so it could already be running, or you should start it by hand later...
 #>
 
 param(
 [Parameter(Mandatory=$true, Position=0)]
 [string]$Application
 ,
 [Parameter(Mandatory=$false)]
 [double]$Interval = 5.0
 ,
 [Parameter(Mandatory=$false)]
 [switch]$Start
 ,
 [Parameter(ValueFromRemainingArguments=$true)]
 [string[]]$Parameters
 )
 
 $ExecutablePath = gcm $Application -type application | select -expand definition -first 1 | %{ $_ -replace '\\','\\' }
 
 if($Parameters) {
    $Parameters = "-ArgumentList `"$($Parameters -join '", "')`""
 }
 
 $sb = Invoke-Expression "{ Start-Process `"$ExecutablePath`" $Parameters }"
 
 Register-WMIEvent -Action $sb -Query "Select * From __InstanceDeletionEvent Within $Interval Where TargetInstance ISA 'Win32_Process' And TargetInstance.ExecutablePath='$ExecutablePath'" | Out-Null
 
 if($Start){ 
    sleep -milli 500
    &$sb 
 }
 
 }
`

