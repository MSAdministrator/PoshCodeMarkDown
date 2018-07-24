---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 1.0
Encoding: ascii
License: cc0
PoshCode ID: 1875
Published Date: 2010-05-26t12
Archived Date: 2016-11-07t09
---

# install-module - 

## Description

a function to install single-file modules (create the folder, make sure the file is named the same as the folder, etc).

## Comments



## Usage



## TODO



## function

`install-module`

## Code

`#
 #
 function Install-Module {
 [CmdletBinding()]
 Param(
     [Parameter(ValueFromPipeline=$true,ValueFromPipelineByPropertyName=$true,Mandatory=$true)]
     [Alias("PSPath")]
     [String]$ModuleFilePath
 ,   [Switch]$Global
 )
     $ModuleFilePath = Resolve-Path $ModuleFilePath
     
     $ModuleName = (ls $ModuleFilePath).BaseName
     
     $PSModulePath = $Env:PSModulePath -split ";" | Select -Index ([int][bool]$Global)
 
     $ModuleFolder = MkDir $PSModulePath\$ModuleName -EA 0 -EV FailMkDir
     if($FailMkDir -and @($FailMkDir)[0].CategoryInfo.Category -eq "PermissionDenied") {
         if($Global) {
             throw "You must be elevated to install a global module."
         } else { throw @($FailMkDir)[0] }
     }
 
     Move-Item $ModuleFilePath $ModuleFolder
 
     Get-Module $ModuleName -List
 <#
 .Synopsis
     Installs a single-file (psm1 or dll) module to the ModulePath
 .Description 
     Supports installing modules for the current user or all users (if elevated)
 .Parameter ModuleFilePath
     The path to the module file to be installed
 .Parameter Global
     If set, attempts to install the module to the all users location in Windows\System32...
 .Example
     Install-Module .\Authenticode.psm1 -Global
 
     Description
     -----------
     Installs the Authenticode module to the System32\WindowsPowerShell\v1.0\Modules for all users to use.
 .Example
     
 
     Description
     -----------
     Uses Get-PoshCode (from the PoshCode module) to download the Impersonation module and then fixes it's file name, and finally installs the Impersonation module for the current user.
 #>
 }
`

