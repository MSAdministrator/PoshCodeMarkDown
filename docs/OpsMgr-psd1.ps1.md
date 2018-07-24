---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: utf-8
License: cc0
PoshCode ID: 2425
Published Date: 
Archived Date: 2011-01-01t01
---

# opsmgr.psd1 - 

## Description

a module manifest wrapper for microsoft’s operations manager shell. requires operations manager shell on build machine, but resulting module can be copied/used on clients w/o operations manager installation.

## Comments



## Usage



## TODO



## module

``

## Code

`#
 #
 if (-not (test-path $home\Documents\WindowsPowerShell\Modules\OpsMgr))
 {new-item  $home\Documents\WindowsPowerShell\Modules\OpsMgr -type directory}
 Set-Location "C:\Program Files\System Center Operations Manager 2007"
 Copy-Item "Microsoft.EnterpriseManagement.OperationsManager.ClientShell.dll","Microsoft.EnterpriseManagement.OperationsManager.ClientShell.Format.ps1xml",`
 "Microsoft.EnterpriseManagement.OperationsManager.ClientShell.Types.ps1xml","Microsoft.EnterpriseManagement.OperationsManager.ClientShell.dll-help.xml",`
 "Microsoft.EnterpriseManagement.OperationsManager.ClientShell.Functions.ps1","Microsoft.EnterpriseManagement.OperationsManager.ClientShell.Startup.ps1",`
 "Microsoft.EnterpriseManagement.OperationsManager.ClientShell.NonInteractiveStartup.ps1" `
 -destination $home\Documents\WindowsPowerShell\Modules\OpsMgr
 Set-Location "C:\Program Files\System Center Operations Manager 2007\SDK Binaries"
 Copy-Item  "Microsoft.EnterpriseManagement.OperationsManager.dll","Microsoft.EnterpriseManagement.OperationsManager.Common.dll" -destination $home\Documents\WindowsPowerShell\Modules\OpsMgr
 #>
 
 
 @{
 ModuleVersion="0.0.0.1"
 Description="A Wrapper for Microsoft's Operations Manager Shell "
 Author="Chad Miller"
 Copyright="© 2010, Chad Miller, released under the Ms-PL"
 CompanyName="http://sev17.com"
 CLRVersion="2.0"
 FormatsToProcess="Microsoft.EnterpriseManagement.OperationsManager.ClientShell.Format.ps1xml"
 NestedModules="Microsoft.EnterpriseManagement.OperationsManager.ClientShell"
 RequiredAssemblies="Microsoft.EnterpriseManagement.OperationsManager.dll","Microsoft.EnterpriseManagement.OperationsManager.Common.dll"
 TypesToProcess="Microsoft.EnterpriseManagement.OperationsManager.ClientShell.Types.ps1xml"
 ScriptsToProcess="Microsoft.EnterpriseManagement.OperationsManager.ClientShell.Functions.ps1"
 }
`

