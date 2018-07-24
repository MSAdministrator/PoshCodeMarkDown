---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 1.0
Encoding: ascii
License: cc0
PoshCode ID: 2692
Published Date: 2011-05-24t20
Archived Date: 2011-05-27t22
---

# qad.psm1 - 

## Description

this is the file you need to load quest’s active directory snap-in as a module.  just put this in a folder with all the dlls …

## Comments



## Usage



## TODO



## module

``

## Code

`#
 #
 #
 #
 #
 #
 
 @{
 
 ModuleToProcess = 'Quest.ActiveRoles.ArsPowerShellSnapIn.dll'
 
 ModuleVersion = '1.0'
 
 GUID = '2b34f8ae-e6eb-4d14-9536-6e69cf4f31ed'
 
 Author = 'Quest.ActiveRoles.ADManagement'
 
 CompanyName = 'Quest Software, Inc.'
 
 Copyright = 'Quest Software, Inc.'
 
 Description = 'This Windows PowerShell snap-in contains cmdlets to manage Active Directory and Quest ActiveRoles Server.'
 
 PowerShellVersion = ''
 
 PowerShellHostName = ''
 
 PowerShellHostVersion = ''
 
 DotNetFrameworkVersion = ''
 
 CLRVersion = ''
 
 ProcessorArchitecture = ''
 
 RequiredModules = @()
 
 RequiredAssemblies = 'Quest.ActiveRoles.ArsPowerShellSnapIn.DirectoryAccess.dll', 
                'Microsoft.Practices.Unity.Interception.dll', 
                'Quest.ActiveRolesServer.Common.dll', 
                'Quest.ActiveRolesServer.Common.Services.dll', 
                'Microsoft.Practices.Unity.dll', 'NLog.dll', 
                'Quest.ActiveRoles.ArsPowerShellSnapIn.Utility.dll', 
                'Interop.ActiveDs.dll', 
                'Microsoft.Practices.Unity.StaticFactory.dll', 
                'Interop.ArsAdsi.dll'
 
 ScriptsToProcess = @()
 
 TypesToProcess = 'Quest.ActiveRoles.ADManagement.Types.ps1xml'
 
 FormatsToProcess = 'Quest.ActiveRoles.ADManagement.Format.ps1xml'
 
 NestedModules = @()
 
 FunctionsToExport = '*'
 
 CmdletsToExport = '*'
 
 VariablesToExport = '*'
 
 AliasesToExport = '*'
 
 ModuleList = @()
 
 FileList = @()
 
 PrivateData = ''
 
 }
`

