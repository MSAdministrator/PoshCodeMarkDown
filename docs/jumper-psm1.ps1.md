---
Author: greg zakharov
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5169
Published Date: 2016-05-20t13
Archived Date: 2016-03-18t21
---

# jumper.psm1 - 

## Description

usage

## Comments



## Usage



## TODO



## class

`push-path`

## Code

`#
 #
 if (!(Test-Path alias:jumper)) { Set-Alias jumper Push-Path }
 
 $asm = Add-Type -MemberDefinition @'
     [DllImport("kernel32.dll")]
     [return: MarshalAs(UnmanagedType.Bool)]
     internal static extern Boolean CloseHandle(IntPtr hObject);
     
     [DllImport("kernel32.dll")]
     internal static extern IntPtr OpenProcess(
         UInt32 dwDesiredAccess,
         [MarshalAs(UnmanagedType.Bool)]Boolean bInheritHandle,
         UInt32 dwProcessId
     );
     
     [DllImport("user32.dll", CharSet = CharSet.Unicode)]
     internal static extern IntPtr FindWindow(
         String lpClassName,
         String lpWindowName
     );
     
     [DllImport("user32.dll", CharSet = CharSet.Unicode)]
     internal static extern IntPtr FindWindowEx(
         IntPtr hwndParent,
         IntPtr hwndChildAfter,
         String lpszClass,
         String lpszWindow
     );
     
     [DllImport("user32.dll")]
     internal static extern UInt32 GetWindowThreadProcessId(
         IntPtr hWnd,
         out UInt32 lpwdProcessId
     );
     
     [DllImport("user32.dll")]
     internal static extern IntPtr SendMessage(
         IntPtr hWnd,
         UInt32 Msg,
         IntPtr wParam,
         IntPtr lParam
     );
     
     [DllImport("user32.dll")]
     internal static extern IntPtr SetFocus(IntPtr hWnd);
     
     [DllImport("user32.dll")]
     [return: MarshalAs(UnmanagedType.Bool)]
     internal static extern Boolean SetForegroundWindow(IntPtr hWnd);
     
     [DllImport("user32.dll")]
     [return: MarshalAs(UnmanagedType.Bool)]
     internal static extern Boolean ShowWindow(
         IntPtr hWnd,
         UInt32 nCmdShow
     );
     
     const UInt32 SW_SHOW     = 0x00000005;
     const UInt32 SYNCHRONIZE = 0x00100000;
     
     public static void RegJump(String path) {
       IntPtr RegEditMain;
       IntPtr RegEditHwnd;
       UInt32 ProcessId = 0;
       IntPtr hndl;
       
       RegEditMain = FindWindow("RegEdit_RegEdit", null);
       if (RegEditMain == IntPtr.Zero) {
         Process.Start("regedit.exe");
         Thread.Sleep(100); //make a pause before next step
         RegEditMain = FindWindow("RegEdit_RegEdit", null);
       }
       
       if (RegEditMain == IntPtr.Zero) {
         Console.WriteLine("Unable to launch regedit.");
         Environment.Exit(1);
       }
       
       ShowWindow(RegEditMain, SW_SHOW);
       RegEditHwnd = FindWindowEx(RegEditMain, IntPtr.Zero, "SysTreeView32", null);
       SetFocus(RegEditHwnd);
       
       if (GetWindowThreadProcessId(RegEditHwnd, out ProcessId) != 0) {
         hndl = OpenProcess(SYNCHRONIZE, false, ProcessId);
         for (Int32 i = 0; i < 30; i++)
           SendMessage(RegEditHwnd, (UInt32)0x100, (IntPtr)0x25, IntPtr.Zero);
         //jump
         Char[] c = path.ToCharArray();
         for (Int32 i = 0; i < c.Length; i++) {
           if (c[i].Equals('\\'))
             SendMessage(RegEditHwnd, (UInt32)0x100, (IntPtr)0x27, IntPtr.Zero);
           else
             SendMessage(RegEditHwnd, (UInt32)0x102, (IntPtr)c[i], IntPtr.Zero);
         }
         SetForegroundWindow(RegEditMain);
         SetFocus(RegEditMain);
         CloseHandle(hndl);
       }
     }
 '@ -Name Jumper -NameSpace NativeMethods -Using System.Diagnostics, System.Threading -PassThru
 
 function Push-Path {
   <#
     .NOTES
         Author: greg zakharov
   #>
   param(
     [Parameter(Mandatory=$true)]
     [String]$Path
   )
   
   begin {
     function Test-PathAndJump([String]$Path) {
       try {
         [void][IO.Directory]::GetAccessControl($Path)
         [void][Diagnostics.Process]::Start('explorer.exe', '/n, ' + $Path)
       }
       catch [Management.Automation.MethodInvocationException] {
         Write-Host Requesting path access is not allowed.
       }
     }
   }
   process {
     if (Test-Path $Path) { Test-PathAndJump $Path }
     else {
       if (Test-Path ($key = 'Registry::' + $Path.ToUpper())) {
         try {
           $key = '\' + (gi -ea 1 $key).Name + '\'
           $asm::RegJump($key)
         }
         catch { $_.Exception.Message }
       }
     }
   }
   end {''}
 }
 
 Export-ModuleMember -Alias jumper -Function Push-Path
`

