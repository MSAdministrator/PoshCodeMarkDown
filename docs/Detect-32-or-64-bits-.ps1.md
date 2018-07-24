---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: utf-8
License: cc0
PoshCode ID: 1450
Published Date: 
Archived Date: 2009-11-13t08
---

#  - 

## Description

detect 32 or 64 bits windows � regardless of wow64 � with the powershell osarchitecture function

## Comments



## Usage



## TODO



## function

``

## Code

`#
 #
 
 Function global:CurrentProcessIsWOW64
 {
     $code = @"
         using System;
         using System.Runtime.InteropServices;
  
         public class Kernel32
         {
             [DllImport("kernel32.dll", SetLastError = true, CallingConvention = CallingConvention.Winapi)]
             [return: MarshalAs(UnmanagedType.Bool)]
             public static extern bool IsWow64Process([In] IntPtr hProcess, [Out] out bool lpSystemInfo);
  
             // This overload returns True if the current process is running on Wow, False otherwise
             public bool CurrentProcessIsWow64()
             {
                 bool retVal;
                 if (!(IsWow64Process(System.Diagnostics.Process.GetCurrentProcess().Handle, out retVal))) { throw new Exception("IsWow64Process() failed"); }
                 return retVal;
             }
         }
 "@
     InvokeCSharp -code $code -class 'Kernel32' -method 'CurrentProcessIsWow64'
 }
  
 Function global:ProcessArchitecture
 {
     switch ([System.IntPtr]::Size)
     {
         4 { 32 }
         8 { 64 }
         default { throw "Unknown Process Architecture: $([System.IntPtr]::Size * 8) bits" }
     }
 }
  
 Function global:OSArchitecture
 {
     if (((ProcessArchitecture) -eq 32) -and (CurrentProcessIsWOW64)) { 64 } else { ProcessArchitecture }
 }
`

