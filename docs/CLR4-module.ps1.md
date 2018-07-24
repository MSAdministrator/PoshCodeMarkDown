---
Author: beefarino
Publisher: 
Copyright: 
Email: 
Version: 4.0
Encoding: ascii
License: cc0
PoshCode ID: 3294
Published Date: 2012-03-20t07
Archived Date: 2016-08-14t13
---

# clr4 module - 

## Description

runs a command in a powershell instance that hosts the clr v4.0.  if the current runspace is already hosting the 4.0 clr, the command is invoked inline.

## Comments



## Usage



## TODO



## function

`start-clr4`

## Code

`#
 #
 function Start-CLR4 {
    
 	[CmdletBinding()]
     
 	param ( [string] $cmd )
 
 
     
     if ($PSVersionTable.CLRVersion.Major -eq 4) 
     {    
 	write-debug 'already running clr 4'
 	invoke-expression $cmd;
 	return
     }
 
     $RunActivationConfigPath = resolve-path ~ | Join-Path -ChildPath .CLR4PowerShell;
     
     write-debug "clr4 config path: $runactivationconfigpath"
 
     if( -not( test-path $runactivationconfigpath ))
     {
 	   New-Item -Path $RunActivationConfigPath -ItemType Container | Out-Null;
     
 
 @"
 <?xml version="1.0" encoding="utf-8" ?>
 <configuration>
   <startup useLegacyV2RuntimeActivationPolicy="true">
   <supportedRuntime version="v4.0"/>
 </startup>
 </configuration>
 "@ | Set-Content -Path $RunActivationConfigPath\powershell.exe.activation_config -Encoding UTF8;
 
     }
     
     $EnvVarName = 'COMPLUS_ApplicationMigrationRuntimeActivationConfigPath';
     [Environment]::SetEnvironmentVariable($EnvVarName, $RunActivationConfigPath);
     
     write-debug "current COMPLUS_ApplicationMigrationRuntimeActivationConfigPath: $env:COMPLUS_ApplicationMigrationRuntimeActivationConfigPath";
 
     & powershell.exe -nologo -command "$cmd";
 }
`

