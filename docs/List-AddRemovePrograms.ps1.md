---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2470
Published Date: 2011-01-21t13
Archived Date: 2017-05-30t18
---

# list addremoveprograms - 

## Description

this script creates a wmi class win32_addremoveprograms (and, on 64bit systems, a win32_addremoveprograms32 for 32bit apps) which are backed by the registry provider.  they can then be queried to list installed apps (and versions) and perform much faster than running the same queries using the powershell registry provider. additionally, they can be used in gpo policies, etc.

## Comments



## Usage



## TODO



## class

``

## Code

`#
 #
 if(!(gwmi -list Win32_AddRemovePrograms)) {
 
 set-content $pwd\RegProgs.mof @'
 instance of __Win32Provider as $Instprov
 {
 Name ="RegProv" ;
 ClsID = "{fe9af5c0-d3b6-11ce-a5b6-00aa00680c3f}" ;
 };
 instance of __InstanceProviderRegistration
 {
 Provider =$InstProv;
 SupportsPut =TRUE;
 SupportsGet =TRUE;
 SupportsDelete =FALSE;
 SupportsEnumeration = TRUE;
 };
 
 [dynamic, provider("RegProv"),
 ProviderClsid("{fe9af5c0-d3b6-11ce-a5b6-00aa00680c3f}"),
 ClassContext("local|HKEY_LOCAL_MACHINE\\SOFTWARE\\Microsoft\\Windows\\CurrentVersion\\Uninstall")]
 class Win32_AddRemovePrograms
 {
 [key]
 string ProdID;
 [PropertyContext("DisplayName")]
 string DisplayName;
 [PropertyContext("Publisher")]
 string Publisher;
 [PropertyContext("DisplayVersion")]
 string Version;
 [PropertyContext("UninstallString")]
 string UninstallString;
 [PropertyContext("EstimatedSize")]
 string EstimatedSizeKb;
 [PropertyContext("InstallDate")]
 string InstallDate;
 };
 '@
 
 if( test-path HKLM:\SOFTWARE\Wow6432Node ) {
 add-content $pwd\RegProgs.mof @'
 [dynamic, provider("RegProv"),
 ProviderClsid("{fe9af5c0-d3b6-11ce-a5b6-00aa00680c3f}"),
 ClassContext("local|HKEY_LOCAL_MACHINE\\SOFTWARE\\Wow6432Node\\Microsoft\\Windows\\CurrentVersion\\Uninstall")]
 class Win32_AddRemovePrograms32
 {
 [key]
 string ProdID;
 [PropertyContext("DisplayName")]
 string DisplayName;
 [PropertyContext("Publisher")]
 string Publisher;
 [PropertyContext("DisplayVersion")]
 string Version;
 [PropertyContext("UninstallString")]
 string UninstallString;
 [PropertyContext("EstimatedSize")]
 string EstimatedSizeKb;
 [PropertyContext("InstallDate")]
 string InstallDate;
 };
 '@
 }
 
 Write-Warning "MOF compiler will be RunAs Administrator ..."
 if(
     (new-object Security.Principal.WindowsPrincipal ([Security.Principal.WindowsIdentity]::GetCurrent())
     ).IsInRole([Security.Principal.WindowsBuiltInRole]::Administrator) 
 ) {
    mofcomp.exe $pwd\RegProgs.mof
 } else {
 }
 }
 
 
 Get-WmiObject Win32_AddRemovePrograms -filter "DisplayName like '%.NET%'" | Sort-Object DisplayName | Format-Table DisplayName, Version
 
 if( test-path HKLM:\SOFTWARE\Wow6432Node ) {
 Get-WmiObject Win32_AddRemovePrograms32 -filter "DisplayName like '%.NET%'" | Sort-Object DisplayName | Format-Table DisplayName, Version
 
 }
 
 
 Get-ItemProperty HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall\* DisplayName,Publisher,DisplayVersion,UninstallString,EstimatedSize,InstallDate -ea 0 | 
    Where-Object { $_.DisplayName -like "*.Net*" } | Sort-Object DisplayName | Format-Table DisplayName, DisplayVersion, EstimatedSize -AutoSize
 
`

