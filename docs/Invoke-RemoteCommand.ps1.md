---
Author: grant carthew
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 3390
Published Date: 2012-05-01t09
Archived Date: 2016-08-05t07
---

# invoke-remotecommand - 

## Description

this function will run code on a remote computer under the currently logged on user credentials. knowing there password is not required as it will impersonate the windows token.

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 <#
  #>
  
  Function Global:Invoke-RemoteCommand {
 
     [CmdletBinding(
         SupportsShouldProcess=$true,
         ConfirmImpact="Low"
 
     Param (
         [Parameter(
             ValueFromPipeline=$true,
             Position=0)]
         [String]$computerName = "localhost",
         
         [Parameter(Position=1)][String]$inputCode,
         
         [Parameter(Position=2)][String]$outputFile = "C:\Temp\Output.ps1",
         [Switch]$passthru
     
     Begin {
         If ($MyInvocation.BoundParameters.Verbose -match $true) {
             $local:VerbosePreference = "Continue"
             $local:ErrorActionPreference = "Continue"
             $local:verbose = $true
         } Else {
             $local:VerbosePreference = "SilentlyContinue"
             $local:ErrorActionPreference = "SilentlyContinue"
             $local:verbose = $false
 
         Function local:Test-AdministratorPrivileges {
     		Write-Verbose "Get user that script context is being run in."
     		$identity = [Security.Principal.WindowsIdentity]::GetCurrent()
     		
     		Write-Verbose "Create a new Security Principal object to retrieve rights."
     		$principal = new-object Security.Principal.WindowsPrincipal $identity
     		
     		Write-Verbose "Retrieve if user is currently in an Administrator role."
     		$result = $principal.IsInRole([Security.Principal.WindowsBuiltInRole]::Administrator)
 		
     		If ($result) {
     			Write-Output $true
     		} Else {
     			Write-Output $false
         
         If (Test-AdministratorPrivileges -Verbose:$verbose) {
             Write-Verbose "Function is being run as an Administrator."
         } Else {
             Write-Host "Current user, {0}, is not an Administrator." `
                 -F ([Security.Principal.WindowsIdentity]::GetCurrent()) `
                 -ForegroundColor Red
             Break
         
         $servicePath = "C:\Temp"
         $serviceName = "ps_service"
 
     Process {
         If ($MyInvocation.BoundParameters.Debug -match $true) {
             $local:DebugPreference = "Continue"
         } Else {
             $local:DebugPreference = "SilentlyContinue"   
     
         $scriptblock = {
             Param(
                 $inputCode,
                 $outputFile,
                 $servicePath,
                 $serviceName
             )            
         
             $results = c:\windows\system32\query.exe session
 
             $active = $results | %{        
                 $line = $_.ToString().Split(" ")
                 $array = $line | Where { $_ -ne "" }
                 
                 Switch ($array.count) {
                     3 {
                         $sessionname = $array[0]
                         $username = $null
                         $id = $array[1]
                         $state = $array[2]
                         $type = $null
                         Break
                     }
                     
                     4 {
                         $sessionname = $array[0]
                         $username = $array[1]
                         $id = $array[2]
                         $state = $array[3]
                         $type = $null
                         Break        
                     }
                     
                     5 {
                         $sessionname = $array[0]
                         $username = $array[1]
                         $id = $array[2]
                         $state = $array[3]
                         $type = $array[4]
                         Break        
                     }
                 
                 $object = New-Object PSObject -Property @{
                     sessionname = $sessionname
                     username = $username
                     ID = $id
                     state = $state
                     type = $type
                 }
                 
                 Write-Output $object
             } | Where { $_.State -eq "Active" }
 
             If ($active) {
                 
                 Write-Verbose $outputFile
                 Write-Verbose $inputCode
                 Out-File -FilePath $outputFile -InputObject $inputCode -Encoding ascii
 
                 $command = "powershell.exe -NonInteractive -WindowStyle hidden -NoLogo -File $outputFile -ExecutionPolicy RemoteSigned"
                 $serviceDisplayName = "PS Emulate Session"
 
                 Write-Verbose $command
                 Write-Verbose $serviceDisplayName
 
                 $source = @"
                 using System;
                 using System.Text;
                 using System.Runtime.InteropServices;
                 using System.ServiceProcess;
                 using System.Diagnostics;
 
                 namespace PS
                 {
                     public class Emulate : System.ServiceProcess.ServiceBase {
                     
                         public Emulate() {
                             this.ServiceName = "$serviceDisplayName";
                         }
                         
                         static void Main(string[] args) {
                             System.ServiceProcess.ServiceBase.Run(new Emulate());
                         }
                         
                         protected override void OnStart(string[] args) {
                             base.OnStart(args);
                             
                             EmulateSession.StartProcessInSession($($active.Id), @"$command");      
                         }
                         
                         protected override void OnStop() {
                             base.OnStop();
                         }
                         
                         public static void WriteToEventLog(string message) {
                             string cs = "PS_Emulate_Service";
                             EventLog elog = new EventLog();
                             if (!EventLog.SourceExists(cs))
                             {
                                 EventLog.CreateEventSource(cs, "Application");
                             }
                             elog.Source = cs;
                             elog.EnableRaisingEvents = true;
                             EventLog.WriteEntry(cs, message, EventLogEntryType.Information);
                         }
                     }
 
                     static public class EmulateSession
                     {
                         /* structs, enums, and external functions defined at end of code */
 
                         public static System.Diagnostics.Process StartProcessInSession(int sessionID, String commandLine)
                         {
                             Emulate.WriteToEventLog("Inside StartProcessInSession");
                             Emulate.WriteToEventLog("Session ID: " + sessionID.ToString());
                             IntPtr userToken;
                             if (WTSQueryUserToken(sessionID, out userToken))
                             {
                                 //note that WTSQueryUserToken only works when in context of local system account with SE_TCB_NAME
                                 IntPtr lpEnvironment;
                                 Emulate.WriteToEventLog("Token: " + userToken.ToString());
                                 if (CreateEnvironmentBlock(out lpEnvironment, userToken, false))
                                 {
                                     Emulate.WriteToEventLog("User Env: " + lpEnvironment.ToString());
                                     StartupInfo si = new StartupInfo();
                                     si.cb = Marshal.SizeOf(si);
                                     si.lpDesktop = "winsta0\\default";
                                     si.dwFlags = STARTF.STARTF_USESHOWWINDOW;
                                     // Using the SW_HIDE will make the window hidden, see in the bottom section for more commands
                                     si.wShowWindow = ShowWindow.SW_HIDE;
                                     ProcessInformation pi;
                                     // Note the CreationFlags, they make this work as it must have both the CREATE_NEW_CONSOLE and CREATE_UNICODE_ENVIRONMENT
                                     if (CreateProcessAsUser(userToken, null, new StringBuilder(commandLine), IntPtr.Zero, IntPtr.Zero, false, CreationFlags.CREATE_NEW_CONSOLE | CreationFlags.CREATE_UNICODE_ENVIRONMENT, lpEnvironment, null, ref si, out pi))
                                     {
                                         CloseHandle(pi.hThread);
                                         CloseHandle(pi.hProcess);
                                         //context.Undo();
                                         try
                                         {
                                             return System.Diagnostics.Process.GetProcessById(pi.dwProcessId);
                                         }
                                         catch (ArgumentException)
                                         {
                                             // I had to remove the ArgumentException e (I removed the e), it threw up a compiler warning
                                             //The process ID couldn't be found - which is what always happens because it has closed
                                             return null;
                                         }
                                     }
                                     else
                                     {
                                         Emulate.WriteToEventLog("Could Not Create Process.");
                                         int err = Marshal.GetLastWin32Error();
                                         throw new System.ComponentModel.Win32Exception(err, "Could not create process.\nWin32 error: " + err.ToString());
                                     }
                                 }
                                 else
                                 {
                                     Emulate.WriteToEventLog("Could not create environment block.");
                                     int err = Marshal.GetLastWin32Error();
                                     throw new System.ComponentModel.Win32Exception(err, "Could not create environment block.\nWin32 error: " + err.ToString());
                                 }
                             }
                             else
                             {
                                 Emulate.WriteToEventLog("No Token");
                                 int err = System.Runtime.InteropServices.Marshal.GetLastWin32Error();
                                 if (err == 1008) return null; //There is no token
                                 throw new System.ComponentModel.Win32Exception(err, "Could not get the user token from session " + sessionID.ToString() + " - Error: " + err.ToString());
                             }
                         }
                         
                         [DllImport("wtsapi32.dll", SetLastError = true)]
                         static extern bool WTSQueryUserToken(Int32 sessionId, out IntPtr Token);
 
                         [DllImport("userenv.dll", SetLastError = true)]
                         static extern bool CreateEnvironmentBlock(out IntPtr lpEnvironment, IntPtr hToken, bool bInherit);
 
                         [DllImport("advapi32.dll", SetLastError = true, CharSet = CharSet.Auto)]
                         static extern bool CreateProcessAsUser(IntPtr hToken, String lpApplicationName, [In] StringBuilder lpCommandLine, IntPtr /*to a SecurityAttributes struct or null*/ lpProcessAttributes, IntPtr /*to a SecurityAttributes struct or null*/ lpThreadAttributes, bool bInheritHandles, CreationFlags creationFlags, IntPtr lpEnvironment, String lpCurrentDirectory, ref StartupInfo lpStartupInfo, out ProcessInformation lpProcessInformation);
 
                         [DllImport("kernel32.dll", SetLastError = true)]
                         static extern bool CloseHandle(IntPtr hHandle);
 
                         [StructLayout(LayoutKind.Sequential, CharSet = CharSet.Unicode)]
                         struct StartupInfo
                         {
                             public Int32 cb;
                             public String lpReserved;
                             public String lpDesktop;
                             public String lpTitle;
                             public Int32 dwX;
                             public Int32 dwY;
                             public Int32 dwXSize;
                             public Int32 dwYSize;
                             public Int32 dwXCountChars;
                             public Int32 dwYCountChars;
                             public Int32 dwFillAttribute;
                             public STARTF dwFlags;
                             public ShowWindow wShowWindow;
                             public Int16 cbReserved2;
                             public IntPtr lpReserved2;
                             public IntPtr hStdInput;
                             public IntPtr hStdOutput;
                             public IntPtr hStdError;
                         }
 
                         [StructLayout(LayoutKind.Sequential)]
                         internal struct ProcessInformation
                         {
                             public IntPtr hProcess;
                             public IntPtr hThread;
                             public int dwProcessId;
                             public int dwThreadId;
                         }
 
 
                         /// <summary>
                         /// The following process creation flags are used by the CreateProcess, CreateProcessAsUser, CreateProcessWithLogonW, and CreateProcessWithTokenW functions. They can be specified in any combination, except as noted.
                         /// </summary>
                         [Flags]
                         enum CreationFlags : int
                         {
                             /// <summary>
                             /// Not specified by MSDN???
                             /// </summary>
                             NONE = 0,
 
                             /// <summary>
                             /// The calling thread starts and debugs the new process and all child processes created by the new process. It can receive all related debug events using the WaitForDebugEvent function. 
                             /// A process that uses DEBUG_PROCESS becomes the root of a debugging chain. This continues until another process in the chain is created with DEBUG_PROCESS.
                             /// If this flag is combined with DEBUG_ONLY_THIS_PROCESS, the caller debugs only the new process, not any child processes.
                             /// </summary>
                             DEBUG_PROCESS = 0x00000001,
 
                             /// <summary>
                             /// The calling thread starts and debugs the new process. It can receive all related debug events using the WaitForDebugEvent function.
                             /// </summary>
                             DEBUG_ONLY_THIS_PROCESS = 0x00000002,
 
                             /// <summary>
                             /// The primary thread of the new process is created in a suspended state, and does not run until the ResumeThread function is called.
                             /// </summary>
                             CREATE_SUSPENDED = 0x00000004,
 
                             /// <summary>
                             /// For console processes, the new process does not inherit its parent's console (the default). The new process can call the AllocConsole function at a later time to create a console. For more information, see Creation of a Console. 
                             /// This value cannot be used with CREATE_NEW_CONSOLE.
                             /// </summary>
                             DETACHED_PROCESS = 0x00000008,
 
                             /// <summary>
                             /// The new process has a new console, instead of inheriting its parent's console (the default). For more information, see Creation of a Console. 
                             /// This flag cannot be used with DETACHED_PROCESS.
                             /// </summary>
                             CREATE_NEW_CONSOLE = 0x00000010,
 
                             /// <summary>
                             /// The new process is the root process of a new process group. The process group includes all processes that are descendants of this root process. The process identifier of the new process group is the same as the process identifier, which is returned in the lpProcessInformation parameter. Process groups are used by the GenerateConsoleCtrlEvent function to enable sending a CTRL+BREAK signal to a group of console processes.
                             /// If this flag is specified, CTRL+C signals will be disabled for all processes within the new process group.
                             /// This flag is ignored if specified with CREATE_NEW_CONSOLE.
                             /// </summary>
                             CREATE_NEW_PROCESS_GROUP = 0x00000200,
 
                             /// <summary>
                             /// If this flag is set, the environment block pointed to by lpEnvironment uses Unicode characters. Otherwise, the environment block uses ANSI characters.
                             /// </summary>
                             CREATE_UNICODE_ENVIRONMENT = 0x00000400,
 
                             /// <summary>
                             /// This flag is valid only when starting a 16-bit Windows-based application. If set, the new process runs in a private Virtual DOS Machine (VDM). By default, all 16-bit Windows-based applications run as threads in a single, shared VDM. The advantage of running separately is that a crash only terminates the single VDM; any other programs running in distinct VDMs continue to function normally. Also, 16-bit Windows-based applications that are run in separate VDMs have separate input queues. That means that if one application stops responding momentarily, applications in separate VDMs continue to receive input. The disadvantage of running separately is that it takes significantly more memory to do so. You should use this flag only if the user requests that 16-bit applications should run in their own VDM.
                             /// </summary>
                             CREATE_SEPARATE_WOW_VDM = 0x00000800,
 
                             /// <summary>
                             /// The flag is valid only when starting a 16-bit Windows-based application. If the DefaultSeparateVDM switch in the Windows section of WIN.INI is TRUE, this flag overrides the switch. The new process is run in the shared Virtual DOS Machine.
                             /// </summary>
                             CREATE_SHARED_WOW_VDM = 0x00001000,
 
                             /// <summary>
                             /// The process is to be run as a protected process. The system restricts access to protected processes and the threads of protected processes. For more information on how processes can interact with protected processes, see Process Security and Access Rights.
                             /// To activate a protected process, the binary must have a special signature. This signature is provided by Microsoft but not currently available for non-Microsoft binaries. There are currently four protected processes: media foundation, audio engine, Windows error reporting, and system. Components that load into these binaries must also be signed. Multimedia companies can leverage the first two protected processes. For more information, see Overview of the Protected Media Path.
                             /// Windows Server 2003 and Windows XP/2000:  This value is not supported.
                             /// </summary>
                             CREATE_PROTECTED_PROCESS = 0x00040000,
 
                             /// <summary>
                             /// The process is created with extended startup information; the lpStartupInfo parameter specifies a STARTUPINFOEX structure.
                             /// Windows Server 2003 and Windows XP/2000:  This value is not supported.
                             /// </summary>
                             EXTENDED_STARTUPINFO_PRESENT = 0x00080000,
 
                             /// <summary>
                             /// The child processes of a process associated with a job are not associated with the job. 
                             /// If the calling process is not associated with a job, this constant has no effect. If the calling process is associated with a job, the job must set the JOB_OBJECT_LIMIT_BREAKAWAY_OK limit.
                             /// </summary>
                             CREATE_BREAKAWAY_FROM_JOB = 0x01000000,
 
                             /// <summary>
                             /// Allows the caller to execute a child process that bypasses the process restrictions that would normally be applied automatically to the process.
                             /// Windows 2000:  This value is not supported.
                             /// </summary>
                             CREATE_PRESERVE_CODE_AUTHZ_LEVEL = 0x02000000,
 
                             /// <summary>
                             /// The new process does not inherit the error mode of the calling process. Instead, the new process gets the default error mode. 
                             /// This feature is particularly useful for multi-threaded shell applications that run with hard errors disabled.
                             /// The default behavior is for the new process to inherit the error mode of the caller. Setting this flag changes that default behavior.
                             /// </summary>
                             CREATE_DEFAULT_ERROR_MODE = 0x04000000,
 
                             /// <summary>
                             /// The process is a console application that is being run without a console window. Therefore, the console handle for the application is not set.
                             /// This flag is ignored if the application is not a console application, or if it is used with either CREATE_NEW_CONSOLE or DETACHED_PROCESS.
                             /// </summary>
                             CREATE_NO_WINDOW = 0x08000000,
                         }
 
                         [Flags]
                         public enum STARTF : uint
                         {
                             STARTF_USESHOWWINDOW = 0x00000001,
                             STARTF_USESIZE = 0x00000002,
                             STARTF_USEPOSITION = 0x00000004,
                             STARTF_USECOUNTCHARS = 0x00000008,
                             STARTF_USEFILLATTRIBUTE = 0x00000010,
                             STARTF_RUNFULLSCREEN = 0x00000020,  // ignored for non-x86 platforms
                             STARTF_FORCEONFEEDBACK = 0x00000040,
                             STARTF_FORCEOFFFEEDBACK = 0x00000080,
                             STARTF_USESTDHANDLES = 0x00000100,
                         }
 
                         public enum ShowWindow : short
                         {
                             SW_HIDE = 0,
                             SW_SHOWNORMAL = 1,
                             SW_NORMAL = 1,
                             SW_SHOWMINIMIZED = 2,
                             SW_SHOWMAXIMIZED = 3,
                             SW_MAXIMIZE = 3,
                             SW_SHOWNOACTIVATE = 4,
                             SW_SHOW = 5,
                             SW_MINIMIZE = 6,
                             SW_SHOWMINNOACTIVE = 7,
                             SW_SHOWNA = 8,
                             SW_RESTORE = 9,
                             SW_SHOWDEFAULT = 10,
                             SW_FORCEMINIMIZE = 11,
                             SW_MAX = 11
                         }
 
                     }
                 }
 "@
 
                 Write-Verbose $source
                 
                 Add-Type -TypeDefinition $source -Language CSharpVersion3 -OutputAssembly ("$servicePath$serviceName.exe") -OutputType ConsoleApplication -ReferencedAssemblies "System.ServiceProcess"
 
                 $computer = "."
                 $class = "Win32_Service"
                 $method = "Create"
                 $mc = [wmiclass]"\\$computer\ROOT\CIMV2:$class"
                 $inparams = $mc.PSBase.GetMethodParameters($method)
                 $inparams.DesktopInteract = $false
                 $inparams.DisplayName = $serviceDisplayName
                 $inparams.ErrorControl = 0
                 $inparams.LoadOrderGroup = $null
                 $inparams.LoadOrderGroupDependencies = $null
                 $inparams.Name = $serviceDisplayName.Replace(" " ,"_")
                 $inparams.PathName = ("$servicePath$serviceName.exe")
                 $inparams.ServiceDependencies = $null
                 $inparams.ServiceType = 16
                 $inparams.StartMode = "Automatic"
                 $inparams.StartName = $null
                 $inparams.StartPassword = $null
                 
                 $result = $mc.PSBase.InvokeMethod($method,$inparams,$null)
 
                 Start-Service -DisplayName $serviceDisplayName
 
                 $baseproc = Get-WMIObject Win32_Process | Where { $_.Name -eq $serviceName }
 
                 While (Get-WMIObject Win32_Process | Where { $_.ParentProcessId -eq $baseproc.ProcessId }) {
                     Sleep 1
 
                 Stop-Service -DisplayName $serviceDisplayName
 
                 C:\Windows\System32\sc.exe delete $serviceDisplayName.Replace(" " ,"_") | Out-Null
 
                 Remove-Item -Path ("$servicePath$serviceName.exe") -Force
                 Remove-Item -Path ("$servicePath$serviceName.pdb") -Force
                 Remove-Item -Path $outputFile -Force
             } Else {
                 Write-Host "No active sessions"
 
         If (Test-Connection $computerName -Quiet) {
             Try {
                 Invoke-Command `
                     -ComputerName $computerName `
                     -ScriptBlock $scriptblock `
                     -ArgumentList $inputCode,$outputFile,$servicePath,$serviceName
             }
             Catch {
                 Write-Warning "Unable to create remote session: $($_.Exception.Message)"
                 Break
 
         .SYNOPSIS 
         Invoke remote code impersonating the currently logged on user.
 
         .PARAMETER computername
         Computer to run the remote code on.
         
         .PARAMETER inputCode
         The code to run on the remote computer.
         
         .PARAMETER outputFile
         The output data from a file.
 
         .INPUTS
         The input is a computer.
 
         .OUTPUTS
         The output is the returned data from 
 
         .EXAMPLE
         C:\PS> $code = "Out-File -FilePath c:\temp\output.txt -InputObject ([Security.Principal.WindowsIdentity]::GetCurrent())"
         C:\PS> Invoke-RemoteCommand -inputCode $code
     #>        
`

