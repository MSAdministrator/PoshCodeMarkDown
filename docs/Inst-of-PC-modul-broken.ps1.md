---
Author: thebigbear
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5733
Published Date: 2015-02-12t09
Archived Date: 2015-02-15t05
---

# inst of pc modul broken? - 

## Description

the installation of poshcode module and the upgrade of it, is broken.

## Comments



## Usage



## TODO



## module

``

## Code

`#
 #
 
 
 1.    When I use cinst PoshCode the installation is successful according to choco but I can't load the modules
 
 
 2.    Using iwr http://PoshCode.org/i -OutF PC.ps1; .\PC; rm .\PC.ps1 results in the error that Import-AtomFeed is unknown
 
 3.    Manually importing PoshCode.psd1 is working without error: Import-Module d:\Data\Apps\_Chocolatey\lib\PoshCode.4.0.1.5\PoshCode.psd1, but executing Find-Module returns this error:
 
 The error is:
     PS C:\WINDOWS\system32> Find-Module shortcut
     Import-Module : The specified module 'D:\Data\Apps_Chocolatey\lib\PoshCode.4.0.1.5\Repositories\NuGet' was not loaded because no valid module file was found in any module directory.
     At D:\Data\Apps_Chocolatey\lib\PoshCode.4.0.1.5\Repository.psm1:96 char:21
 
         $Command = Import-Module "${PoshCodeModuleRoot}\Repositories\$(${Repo}. ...
         ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
             CategoryInfo : ResourceUnavailable: (D:\Data\Apps_C...ositories\NuGet:String) [Import-Module], FileNotFoundException
             FullyQualifiedErrorId : Modules_ModuleNotFound,Microsoft.PowerShell.Commands.ImportModuleCommand
 
 And indeed, no such module exists.
 
 But same errors also happen IF I DO NOT use choco. PC module just won't install or upgrade.
`

