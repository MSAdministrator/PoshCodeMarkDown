---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6084
Published Date: 2016-11-09t10
Archived Date: 2016-05-17t10
---

# suspend-process - 

## Description

a couple of functions to suspend and resume applications (like what you do with resource monitor).

## Comments



## Usage



## TODO



## function

`suspend-process`

## Code

`#
 #
 Add-Type -Name Threader -Namespace "" -Member @"
    [Flags]
    public enum ThreadAccess : int
    {
       Terminate = (0x0001),
       SuspendResume = (0x0002),
       GetContext = (0x0008),
       SetContext = (0x0010),
       SetInformation = (0x0020),
       GetInformation = (0x0040),
       SetThreadToken = (0x0080),
       Impersonate = (0x0100),
       DirectImpersonation = (0x0200)
    }
    [Flags]
    public enum ProcessAccess : uint
    {
       Terminate = 0x00000001,
       CreateThread = 0x00000002,
       VMOperation = 0x00000008,
       VMRead = 0x00000010,
       VMWrite = 0x00000020,
       DupHandle = 0x00000040,
       SetInformation = 0x00000200,
       QueryInformation = 0x00000400,
       SuspendResume = 0x00000800,
       Synchronize = 0x00100000,
       All = 0x001F0FFF
    }
 
    [DllImport("ntdll.dll", EntryPoint = "NtSuspendProcess", SetLastError = true)]
    public static extern uint SuspendProcess(IntPtr processHandle);
    
    [DllImport("ntdll.dll", EntryPoint = "NtResumeProcess", SetLastError = true)]
    public static extern uint ResumeProcess(IntPtr processHandle);
    
    [DllImport("kernel32.dll")]
    public static extern IntPtr OpenProcess(ProcessAccess dwDesiredAccess, bool bInheritHandle, uint dwProcessId);
 
    [DllImport("kernel32.dll")]
    public static extern IntPtr OpenThread(ThreadAccess dwDesiredAccess, bool bInheritHandle, uint dwThreadId);
    
    [DllImport("kernel32.dll", SetLastError=true)]
    public static extern bool CloseHandle(IntPtr hObject);
 
    [DllImport("kernel32.dll")]
    public static extern uint SuspendThread(IntPtr hThread);
 
    [DllImport("kernel32.dll")]
    public static extern int ResumeThread(IntPtr hThread);
 "@
 
 
 
 function Suspend-Process {
 param(
 [Parameter(ValueFromPipeline=$true,Mandatory=$true)]
 [System.Diagnostics.Process]
 $Process
 )
 process {
    
    if(($pProc = [Threader]::OpenProcess("SuspendResume", $false, $Process.Id)) -ne [IntPtr]::Zero) {
       Write-Verbose "Suspending Process: $pProc"
       $result = [Threader]::SuspendProcess($pProc)
       if($result -ne 0) {
          Write-Error "Failed to Suspend: $result"
       }
       [Threader]::CloseHandle($pProc)
    } else {
       Write-Error "Unable to open Process $($Process.Id), are you running elevated?"
    }
 }
 }
 function Resume-Process {
 param(
 [Parameter(ValueFromPipeline=$true,Mandatory=$true)]
 [System.Diagnostics.Process]
 $Process
 )
 process {
    if(($pProc = [Threader]::OpenProcess("SuspendResume", $false, $Process.Id)) -ne [IntPtr]::Zero) {
       Write-Verbose "Resuming Process: $pProc"
       $result = [Threader]::ResumeProcess($pProc)
       if($result -ne 0) {
          Write-Error "Failed to Suspend: $result"
       }
       [Threader]::CloseHandle($pProc)
    } else {
       Write-Error "Unable to open Process $($Process.Id), are you running elevated?"
    }
 }
 }
 
 
 function Suspend-Thread {
 param(
 [Parameter(ValueFromPipeline=$true,ValueFromPipelineByPropertyName=$true,Mandatory=$true)]
 [System.Diagnostics.ProcessThread[]]
 [Alias("Threads")]
 $Thread
 )
 process {
    if(($pThread = [Threader]::OpenThread("SuspendResume", $false, $Thread.Id)) -ne [IntPtr]::Zero) {
       Write-Verbose "Suspending Thread: $pThread"
       [Threader]::SuspendThread($pThread)
       [Threader]::CloseHandle($pThread)
    } else {
       Write-Error "Unable to open Thread $($Thread.Id), are you running elevated?"
    }
 }
 }
 function Resume-Thread {
 param(
 [Parameter(ValueFromPipeline=$true,ValueFromPipelineByPropertyName=$true,Mandatory=$true)]
 [System.Diagnostics.ProcessThread[]]
 [Alias("Threads")]
 $Thread
 )
 process {
    if(($pThread = [Threader]::OpenThread("SuspendResume", $false, $Thread.Id)) -ne [IntPtr]::Zero) {
       Write-Verbose "Resuming Thread: $pThread"
       [Threader]::ResumeThread($pThread)
       [Threader]::CloseHandle($pThread)
    } else {
       Write-Error "Unable to open Thread $($Thread.Id), are you running elevated?"
    }
 }
 }
`

