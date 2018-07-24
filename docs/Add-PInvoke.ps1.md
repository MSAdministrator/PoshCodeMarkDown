---
Author: anonymous
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 4290
Published Date: 2014-07-03t10
Archived Date: 2017-05-22t01
---

# add-pinvoke.ps1 - 

## Description

this file was uploaded by a powergui script editor add-on.

## Comments



## Usage



## TODO



## function

`add-pinvoke`

## Code

`#
 #
 function Add-PInvoke
 {
     <#
     .Synopsis
     .Description
     .Example
         Add-PInvoke -Library User32.dll -Signature GetSystemMetrics(uint Metric)
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
         HelpMessage="The Signature of the Method, i.e. int GetSystemMetrics(int Metric")]
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
         
         $MemberDefinition = "[DllImport(`"$Library`")]
 $Signature"
         
         if (-not $OutputText) {
             Add-Type -PassThru -Name "PInvoke$(Get-Random)" `
                 -MemberDefinition $MemberDefinition
         } else {
             $MemberDefinition
         }
     }
 }
`

