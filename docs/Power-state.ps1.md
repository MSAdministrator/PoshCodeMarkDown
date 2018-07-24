---
Author: greg zakharov
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 3984
Published Date: 2013-02-22t15
Archived Date: 2013-02-28t00
---

# power state - 

## Description

this is old code on c# (written on v2). the code demonstrates how to put the computer into standby, hibernation, how to restart and turn it off (and logoff).

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 using System;
 using System.IO;
 using System.Reflection;
 using System.Diagnostics;
 using System.Globalization;
 using System.Runtime.InteropServices;
 
 [assembly: AssemblyCompany("greg zakharov")]
 [assembly: AssemblyCopyright("Copyleft (C) 2007 greg zakharov")]
 [assembly: AssemblyCulture("")]
 [assembly: AssemblyDescription("Power management utility")]
 [assembly: AssemblyProduct("pwrman.exe")]
 [assembly: AssemblyTitle("Power management utility")]
 [assembly: AssemblyVersion("2.0.0.0")]
 
 namespace PowerManagement {
   internal class AssemblyInfo {
     private Type asm;
     public AssemblyInfo() { asm = typeof(Program); }
 
     public string Copyleft {
       get {
         Type atr = typeof(AssemblyCopyrightAttribute);
         object[] arr = asm.Assembly.GetCustomAttributes(atr, false);
         AssemblyCopyrightAttribute cr = (AssemblyCopyrightAttribute) arr[0];
         return cr.Copyright;
       }
     }
 
     public string Description {
       get {
         Type atr = typeof(AssemblyDescriptionAttribute);
         object[] arr = asm.Assembly.GetCustomAttributes(atr, false);
         AssemblyDescriptionAttribute des = (AssemblyDescriptionAttribute) arr[0];
         return des.Description;
       }
     }
 
     public string Module {
       get { return Path.GetFileName(asm.Assembly.Location); }
     }
 
     public string Title {
       get { return asm.Assembly.GetName().Name; }
     }
 
     public string Version {
       get { return asm.Assembly.GetName().Version.ToString(2); }
     }
   }
 
   internal static class NativeMethods {
     public enum ExitWinCmd : uint {
       EWX_LOGOFF      = 0x00000000,
       EWX_SHUTDOWN    = 0x00000001,
       EWX_REBOOT      = 0x00000002,
       EWX_FORCE       = 0x00000004
     }
 
     [StructLayout(LayoutKind.Sequential, Pack = 1)]
     internal struct TokPriv1Luid {
       public int Count;
       public long Luid;
       public int Attrb;
     }
 
     [DllImport("advapi32.dll")]
     [return: MarshalAs(UnmanagedType.Bool)]
     internal static extern bool AdjustTokenPrivileges(IntPtr TokenHandle,
                 [MarshalAs(UnmanagedType.Bool)]bool DisableAllPrivileges,
                             ref TokPriv1Luid NewState, uint BufferLength,
                               IntPtr PreviousState, IntPtr ReturnLength);
 
     [DllImport("advapi32.dll", CharSet = CharSet.Unicode)]
     [return: MarshalAs(UnmanagedType.Bool)]
     internal static extern bool LookupPrivilegeValue(string lpSystemName,
                                          string lpName, ref long lpLuid);
 
     [DllImport("advapi32.dll")]
     [return: MarshalAs(UnmanagedType.Bool)]
     internal static extern bool OpenProcessToken(IntPtr ProcessHandle,
                           uint DesiredAccess, ref IntPtr TokenHandle);
 
     [DllImport("kernel32.dll")]
     internal static extern IntPtr GetCurrentProcess();
 
     [DllImport("powrprof.dll")]
     [return: MarshalAs(UnmanagedType.U1)]
     internal static extern bool SetSuspendState(
       [In, MarshalAs(UnmanagedType.U1)]bool Hibernate,
       [In, MarshalAs(UnmanagedType.U1)]bool ForceCritical,
       [In, MarshalAs(UnmanagedType.U1)]bool DisableWakeEvent
     );
 
     [DllImport("user32.dll")]
     [return: MarshalAs(UnmanagedType.Bool)]
     internal static extern bool ExitWindowsEx(ExitWinCmd uFlags, uint dwReason);
 
     [DllImport("user32.dll")]
     [return: MarshalAs(UnmanagedType.Bool)]
     internal static extern bool LockWorkStation();
     //
     //combine calls into shared functions
     //
     internal static void ChangeSystemState(ExitWinCmd flags) {
       TokPriv1Luid tpl;
       IntPtr hdl = GetCurrentProcess();
       IntPtr tkn = IntPtr.Zero;
       //TOKEN_ADJUST_PRIVELEGES = 0x00000020
       //TOKEN_QUERY             = 0x00000008
       OpenProcessToken(hdl, (uint)32 | (uint)8, ref tkn);
       tpl.Count = 1;
       tpl.Luid  = 0;
       tpl.Attrb = 2; //SE_PRIVILEGE_ENABLED
       LookupPrivilegeValue(null, "SeShutdownPrivilege", ref tpl.Luid);
       AdjustTokenPrivileges(tkn, false, ref tpl, 0, IntPtr.Zero, IntPtr.Zero);
       //SHTDN_REASON_MAJOR_OTHER = 0x00000000
       //SHTDN_REASON_MINOR_OTHER = 0x00000000
       ExitWindowsEx(flags, (uint)0 | (uint)0);
     }
 
     internal static void Logoff() {
       ChangeSystemState(ExitWinCmd.EWX_LOGOFF | ExitWinCmd.EWX_FORCE);
     }
 
     internal static void Shutdown() {
       ChangeSystemState(ExitWinCmd.EWX_SHUTDOWN | ExitWinCmd.EWX_FORCE);
     }
 
     internal static void Reboot() {
       ChangeSystemState(ExitWinCmd.EWX_REBOOT | ExitWinCmd.EWX_FORCE);
     }
   }
 
   internal sealed class Program {
     static void Main(string[] args) {
       try {
         //argument should always starts with "-" or "/"
         char chr = args[0].ToCharArray()[0];
         //validate delimiter
         if (chr == '-' || chr == '/') {
           //validate argument
           string par = args[0].ToLower(CultureInfo.CurrentCulture).TrimStart('-', '/');
 
           switch (par) {
             case "h": NativeMethods.SetSuspendState(true, true, true); break;
             case "l": NativeMethods.LockWorkStation(); break;
             case "o": NativeMethods.Logoff(); break;
             case "r": NativeMethods.Reboot(); break;
             case "s": NativeMethods.Shutdown(); break;
             case "w": NativeMethods.SetSuspendState(false, true, true); break;
             case "?": PrintHelpInfo(); break;
             default: Console.WriteLine("Error: unknown parameter {0}.", args[0]); break;
           }
         }
         else Console.WriteLine("Invalid parameter. Use \"/?\" for additional info.");
       }
       catch (IndexOutOfRangeException e) { Console.WriteLine(e.Message); }
     }
 
     static void PrintHelpInfo() {
       AssemblyInfo ai = new AssemblyInfo();
 
       Console.WriteLine("{0} v{1} - {2}", ai.Title, ai.Version, ai.Description);
       Console.WriteLine(ai.Copyleft);
       Console.WriteLine("\nUsage: {0} [-h][-l][-o][-r][-s][-w]", ai.Module);
       Console.WriteLine("  -h - hibernate mode");
       Console.WriteLine("  -l - lock workstation");
       Console.WriteLine("  -o - logoff");
       Console.WriteLine("  -r - reboot system");
       Console.WriteLine("  -s - shutdown system");
       Console.WriteLine("  -w - standby mode");
       Console.WriteLine("  -? - this message");
     }
   }
 }
`

