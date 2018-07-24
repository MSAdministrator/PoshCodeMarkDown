---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 4606
Published Date: 2013-11-12t20
Archived Date: 2013-11-14t15
---

# consolehelper.psm1 - 

## Description

some helper functions for getting/setting the output buffer and command history size in powershell.exe native.

## Comments



## Usage



## TODO



## class

`set-historyconfig`

## Code

`#
 #
 Add-Type @"
 
 namespace Win32 {
   using System;
   using System.ComponentModel;
   using System.Runtime.InteropServices;
   
   public struct ShortPoint
   {
     public short X;
     public short Y;
   }
   
   public struct ShortRect
   {
     public short Left;
     public short Top;
     public short Right;
     public short Bottom;
   }
   
   public enum StandardHandle {
     Input = -10,
     Output = -11,
     Error = -12
   }
 
   public static class Console {
 
     public struct BufferInfo
     {
       public ShortPoint Size;
       public ShortPoint CursorPosition;
       public short Attributes;
       public ShortRect Window;
       public ShortPoint MaximumWindowSize;
     }
 
     public struct HistoryInfo {
       public uint cbSize;
       public uint BufferSize;
       public uint BufferCount;
       public uint dwTrimDuplicates;
 
       public bool TrimDuplicates { 
         get { return dwTrimDuplicates == 1; }
         set { dwTrimDuplicates = (uint)(value ? 1 : 0); }
       }
 
       public HistoryInfo(uint bufferSize, uint bufferCount = 4, bool trimDuplicates = false) {
         cbSize = (uint)Marshal.SizeOf(typeof(HistoryInfo));
         BufferSize = bufferSize;
         BufferCount = bufferCount;
         dwTrimDuplicates = (uint)(trimDuplicates ? 1 : 0);
       }
     }
 
     public static BufferInfo GetBufferInfo() {
       BufferInfo buffer;
       if(GetConsoleScreenBufferInfo( GetStdHandle( StandardHandle.Output ), out buffer )) {
         return buffer;
       } else {
         throw new Win32Exception();
       }
     }
 
     public static void SetBufferSize(short width, short height) {
       ShortPoint size = new ShortPoint { X = width, Y = height };
       if(!SetConsoleScreenBufferSize( GetStdHandle( StandardHandle.Output ), size )) {
         throw new Win32Exception();
       }
     }
   
     public static void SetBufferSize(short height) {
       BufferInfo buffer = GetBufferInfo();
       ShortPoint size = new ShortPoint { X = buffer.Size.X, Y = height };
       if(!SetConsoleScreenBufferSize( GetStdHandle( StandardHandle.Output ), size )) {
         throw new Win32Exception();
       }
     }
 
     public static HistoryInfo GetHistoryInfo() {
       HistoryInfo history = new HistoryInfo( 1024 );
       if(GetConsoleHistoryInfo( out history )) {
         return history;
       } else {
         throw new Win32Exception();
       }
     }
 
     public static void SetHistoryInfo(HistoryInfo history) {
       if(!SetConsoleHistoryInfo( history )) {
         throw new Win32Exception();
       }
     }
 
     public static void SetHistoryInfo(uint bufferSize, uint bufferCount = 4, bool trimDuplicates = false ) {
       HistoryInfo history = new HistoryInfo ( bufferSize, bufferCount, trimDuplicates );
 
       if(!SetConsoleHistoryInfo( history )) {
         throw new Win32Exception();
       }
     }
 
 
     [DllImport("kernel32.dll")]
     internal static extern bool GetConsoleScreenBufferInfo(IntPtr hConsoleOutput, out BufferInfo lpConsoleScreenBufferInfo);
     
     [DllImport("kernel32.dll", SetLastError = true)]
     internal static extern bool SetConsoleScreenBufferSize ( IntPtr hConsoleOutput, ShortPoint dwSize );    
     
     [DllImport("kernel32.dll", SetLastError=true)]
     internal static extern IntPtr GetStdHandle(StandardHandle nStdHandle);
 
     [DllImport("kernel32.dll", SetLastError = true)]
     internal static extern bool GetConsoleHistoryInfo( out HistoryInfo ConsoleHistoryInfo );
 
     [DllImport("kernel32.dll", SetLastError = true)]
     internal static extern bool SetConsoleHistoryInfo( HistoryInfo ConsoleHistoryInfo );
   }
 
 }
 "@
 
 
 Update-TypeData -TypeName "Win32.Console+HistoryInfo" -DefaultDisplayPropertySet BufferSize, BufferCount, TrimDuplicates
 
 
 function Set-HistoryConfig {
   [CmdletBinding()]
   param(
     [Parameter(Mandatory=$true)]
     [Int16]$bufferSize, 
     [Int16]$bufferCount = 4, 
     [switch]$trimDuplicates
   )
   [Win32.Console]::SetHistoryInfo( $bufferSize, $bufferCount, $trimDuplicates )
 }
 
 function Get-HistoryConfig {
   [CmdletBinding()]
   param()
   [Win32.Console]::GetHistoryInfo()
 }
 
 
 function Set-BufferSize {
   [CmdletBinding()]
   param(
     [Parameter(Mandatory=$true)]
     [Int16]$height, 
     [Int16]$width
   )
   if($width) {
     [Win32.Console]::SetBufferSize( $width, $height )
   } else {
     [Win32.Console]::SetBufferSize( $height )
   }
 }
 
 function Get-BufferInfo {
   [CmdletBinding()]
   param()
   [Win32.Console]::GetBufferInfo()
 }
 
 function Get-BufferSize {
   [CmdletBinding()]
   param()
   [Win32.Console]::GetBufferInfo().Size
 }
 
 function Get-MaxConsoleSize {
   [CmdletBinding()]
   param()
   [Win32.Console]::GetBufferInfo().MaximumWindowSize
 }
`

