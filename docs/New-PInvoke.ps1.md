---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 1409
Published Date: 2010-10-20t10
Archived Date: 2017-05-30t18
---

# new-pinvoke - 

## Description

a fixed version of the new-pinvoke function that’s in the windows 7 resource kit (powershellpack) pscodegen module.  this one has correct documentation and a better example. it also generates a (global) powershell wrapper function so you can call the api function easily.

## Comments



## Usage



## TODO



## function

`new-pinvoke`

## Code

`#
 #
 function New-PInvoke
 {
     <#
     .Synopsis
         Generate a powershell function alias to a Win32|C api function
     .Description
     .Example
         New-PInvoke user32 "void FlashWindow(IntPtr hwnd, bool bInvert)"
         
 Generates a function for FlashWindow which ignores the boolean return value, and allows you to make a window flash to get the user's attention. Once you've created this function, you could use this line to make a PowerShell window flash at the end of a long-running script:
 
         C:\PS>FlashWindow (ps -id $pid).MainWindowHandle $true
     .Parameter Library
         A C Library containing code to invoke
     .Parameter Signature
     .Parameter OutputText
         If Set, retuns the source code for the pinvoke method.
         If not, compiles the type. 
     #>
     param(
     [Parameter(Mandatory=$true, 
         HelpMessage="The C Library Containing the Function, i.e. User32")]
     [String]
     $Library,
     
     [Parameter(Mandatory=$true,
         HelpMessage="The Signature of the Method, i.e.: int GetSystemMetrics(uint Metric)")]
     [String]
     $Signature,
     
     [Switch]
     $OutputText
     )
     
     process {
         if ($Library -notlike "*.dll*") {
             $Library+=".dll"
         }
         if ($signature -notlike "*;") {
             $Signature+=";"
         }
         if ($signature -notlike "public static extern*") {
             $signature = "public static extern $signature"
         }
         
         $name = $($signature -replace "^.*?\s(\w+)\(.*$",'$1')
         
         $MemberDefinition = "[DllImport(`"$Library`")]`n$Signature"
         
         if (-not $OutputText) {
             $type = Add-Type -PassThru -Name "PInvoke$(Get-Random)" -MemberDefinition $MemberDefinition
             iex "New-Item Function:Global:$name -Value { [$($type.FullName)]::$name.Invoke( `$args ) }"
         } else {
             $MemberDefinition
         }
     }
 }
`

