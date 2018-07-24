---
Author: george howarth
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2018
Published Date: 2011-07-23t04
Archived Date: 2016-03-06t06
---

# test-64bitos - 

## Description

this function will determine whether the local operating system architecture is 64-bit. this has been converted to powershell from the original c# code produced by stefan schultze

## Comments



## Usage



## TODO



## class

`test-64bitos`

## Code

`#
 #
 Add-Type -TypeDefinition @"
 using System;
 using System.Runtime.InteropServices;
 
 public static class Win32Api
 {
     [DllImport("Kernel32.dll", EntryPoint = "IsWow64Process")]
     [return: MarshalAs(UnmanagedType.Bool)]
     public static extern bool IsWow64Process(
         [In] IntPtr hProcess,
         [Out, MarshalAs(UnmanagedType.Bool)] out bool wow64Process
     );
 }
 "@
 
 <#
     .SYNOPSIS
         Determines whether the architecture of the local operating system is 64-bit.
 
     .DESCRIPTION
         The Test-64BitOS function uses the IsWow64Process function to determine whether the process is running under WOW64. It also checks whether the process is 64-bit in this test.
 
     .INPUTS
         None
             This function does not accept any input.
 
     .OUTPUTS
         None
             This function does not return any output.
 
     .NOTES
         Using this function is more reliable than other methods as the function uses a method provided by the operating system to determine whether the process is running under WOW64.
                     
     .EXAMPLE
         C:\PS> Test-64BitOS
         
         This example shows how to use the Test-64BitOS function.
         
     .LINK
         
 #>
 function Test-64BitOS
 {
     return (Test-64BitProcess) -or (Test-Wow64)
 }
 
 function Test-64BitProcess
 {
     return [IntPtr]::Size -eq 8
 }
 
 function Test-Wow64
 {
     if ([Environment]::OSVersion.Version.Major -eq 5 -and 
         [Environment]::OSVersion.Version.Major -ge 1 -or 
         [Environment]::OSVersion.Version.Major -ge 6)
     {
         $process = [System.Diagnostics.Process]::GetCurrentProcess()
         
         $wow64Process = $false
         
         if (![Win32Api]::IsWow64Process($process.Handle, [ref]$wow64Process))
         {
             return $false
         }
         
         return $wow64Process
     }
     else
     {
         return $false
     }
 }
 
 Test-64BitOS
`

