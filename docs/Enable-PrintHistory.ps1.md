---
Author: kevmar
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5191
Published Date: 2014-05-24t04
Archived Date: 2014-05-26t06
---

# enable-printhistory - 

## Description

this enables the microsoft-windows-printservice/operational in the event log. every time something is printed, details about the print job will be recorded into that event log.

## Comments



## Usage



## TODO



## script

`enable-printhistory`

## Code

`#
 #
 <#
 .Synopsis
    Enables logging of print jobs
 .DESCRIPTION
    This enables the Microsoft-Windows-PrintService/Operational in the event log. Every time something is printed, details about the print job will be recorded into that event log.
 .EXAMPLE
    Enable-PrintHistory
 
 #>
 function Enable-PrintHistory
 {
 
     [CmdletBinding()]
     Param
     (
     )
 
     Begin
     {
     }
     Process
     {
         Write-Verbose "Check for administrator rights"
         if( ([Security.Principal.WindowsPrincipal] [Security.Principal.WindowsIdentity]::GetCurrent()).IsInRole([Security.Principal.WindowsBuiltInRole] "Administrator")){
            
             Write-Verbose "Retreiving Microsoft-Windows-PrintService/Operational Object"
             $EventLog = Get-WinEvent -ListLog Microsoft-Windows-PrintService/Operational 
             Write-Verbose ("Current Values: IsEnabled = {0}; MaximumSize = {1}MB; LogMode = {2}" -f $EventLog.IsEnabled,($EventLog.MaximumSizeInBytes / 1MB),$EventLog.LogMode)
               
             Write-Verbose "Enabling Microsoft-Windows-PrintService/Operational event log"
             Write-Verbose "Setting Values: IsEnabled = True; MaximumSizeInBytes=50MB; LogMode = AutoBackup"
             $EventLog | 
                 %{$_.IsEnabled = $true; 
                     $_.MaximumSizeInBytes=50MB;
                     $_.LogMode = "AutoBackup"
                     $_.saveChanges()}
         }else{
             Write-Error "This action requires administrator rights to modify the eventlog"
         }
 
     }
     End
     {
     }
 }
`

