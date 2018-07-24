---
Author: lee holmes
Publisher: 
Copyright: 
Email: 
Version: 8.0
Encoding: ascii
License: cc0
PoshCode ID: 3591
Published Date: 2012-08-23t00
Archived Date: 2016-11-30t18
---

# get-processprofile - 

## Description

uses cdb.exe from the debugging tools for windows to create a sample-based

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 ##############################################################################
 ##
 ##
 ##############################################################################
 
 <#
 
 .SYNOPSIS
 
 Uses cdb.exe from the Debugging Tools for Windows to create a sample-based
 profile of .NET or native applications.
 
 .EXAMPLE
 
 $frames = C:\temp\Get-ProcessProfile.ps1 -ProcessId 11844
 $frames | % { $_[0] } | group | sort Count | Select Count,Name | ft -auto
 
 Runs a sampling profile on process ID 1184. Then, it extracts out the top
 (current) stack entry from each call frame and groups it by the resulting
 text.
 
 This gives a report like the following, which was taken while PowerShell
 version 2 was slowly enumerating a network share. The output below
 demonstrates that PowerShell was spending the majority of its time invoking a
 pipeline, and calling the .NET System.IO.FillAttributeInfo API:
 
 Count Name
 ----- ----
     1 System.Collections.Specialized.HybridDictionary.set_Item(System.Object...
     1 System.Text.StringBuilder..ctor(System.String, Int32, Int32, Int32)
     1 System.Management.Automation.Provider.CmdletProvider.WrapOutputInPSObj...
     1 System.Management.Automation.Provider.NavigationCmdletProvider.GetPare...
     1 System.Management.Automation.Provider.CmdletProvider.get_Force()
     1 System.Management.Automation.Cmdlet.WriteObject(System.Object)
     1 System.String.AppendInPlace(Char[], Int32, Int32, Int32)
     1 Microsoft.PowerShell.ConsoleHostRawUserInterface.LengthInBufferCells(C...
     1 System.Security.Util.StringExpressionSet.CanonicalizePath(System.Strin...
     1 Microsoft.PowerShell.ConsoleControl.GetConsoleScreenBufferInfo(Microso...
     1 System.IO.DirectoryInfo..ctor(System.String, Boolean)
     1 System.Security.Permissions.FileIOPermission.AddPathList(System.Securi...
     2 System.IO.Path.InternalCombine(System.String, System.String)
     2 System.Runtime.InteropServices.SafeHandle.Dispose(Boolean)
    18 System.IO.Directory.InternalGetFileDirectoryNames(System.String, Syste...
    66 System.IO.File.FillAttributeInfo(System.String, WIN32_FILE_ATTRIBUTE_D...
   100 System.Management.Automation.Runspaces.PipelineBase.Invoke(System.Coll...
 
 #>
 
 
 param(
     [Parameter(Mandatory = $true)]
     $ProcessId,
 
     $SampleCount = 100,
 
     [Switch] $UseNativeDebugging,
 
     $CdbPath
 )
 
 if(-not $CdbPath)
 {
     $cdbPaths = "C:\Program Files (x86)\Windows Kits\8.0\Debuggers\x64\cdb.exe",
         "C:\Program Files (x86)\Windows Kits\8.0\Debuggers\x86\cdb.exe",
         "C:\Program Files\Debugging Tools for Windows (x64)\cdb.exe",
         "C:\Program Files\Debugging Tools for Windows (x86)\cdb.exe"
         
     foreach($path in $CdbPaths)
     {
         if(Test-Path $path)
         {
             $CdbPath = $path
             break
         }
     }
 
     if(-not $CdbPath)
     {
         throw "Could not find cdb.exe from the Debugging Tools for Windows package"
     }
 }
 
 if(-not (Get-Process -Id $ProcessId))
 {
     throw "Could not find process ID $ProcessId"
 }
 
 $debuggingCommand = ""
 $managedDebuggingCommand = ".loadby sos mscorwks; .loadby sos clr; ~*e !CLRStack"
 $nativeDebuggingCommand = "~*k"
 
 if($UseNativeDebugging)
 {
     $debuggingCommand = $nativeDebuggingCommand
 }
 else
 {
     $debuggingCommand = $managedDebuggingCommand
 }
 
 $startInfo = [System.Diagnostics.ProcessStartInfo] @{
     FileName = $CdbPath;
     Arguments = "-p $processId -noinh -c `"$debuggingCommand`"";
     UseShellExecute = $false;
     RedirectStandardInput = $true
     RedirectStandardOutput = $true
 }
 
 $frames = @()
 
 1..$SampleCount | % {
     $process = [System.Diagnostics.Process]::Start($startInfo)
     $process.StandardInput.WriteLine("qd")
     $process.StandardInput.Close()
     $r = $process.StandardOutput.ReadToEnd() -split "`n"
 
     $frame = @()
     $capture = $false
     switch -regex ($r)
     {
         'Child SP|Child-SP' { $capture = $true; continue; }
         'OS Thread Id|^\s*$' { $capture = $false; if($frame) { $frames += ,$frame; $frame = @() } }
         { $capture -and ($_ -match '\)$|!') } { $frame += $_ -replace ".*? .*? ([^+]*).*",'$1' }
     }
 
     if($frame) { $frames += ,$frame }
 
     Start-Sleep -m (100 + (Get-Random 100))
 }
 
 ,$frames
`

