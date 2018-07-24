---
Author: thell fowler
Publisher: 
Copyright: 
Email: 
Version: 1.0.1.0
Encoding: ascii
License: cc0
PoshCode ID: 1608
Published Date: 
Archived Date: 2010-02-03t15
---

# loadmoduleconfig - 

## Description

module aware local user configuration settings generator, loader, and exporter setup that invokes during module import.  unlike a module’s privatedata this was created for storing user/project settings that wouldn’t change with module versions (like default param values the user would get tired of passing).  and instead of doing xml creation and parsing manually it makes use of import/export clixml.

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 ################################################################################
 ##
 ##
 ##
 ################################################################################
 
 <#
 	To make use of this script use new-modulemanifest to create a manifest named
 	the with the same name as the manifest you are loading data for but with an
 	extension of 'config.psd1'
 	
 	Enter LoadModuleConfig.ps1 in the NestedModules field.
 	
 	After creating the manifest open it in your editor and edit the PrivateData
 	field to include hashes for LocalUser and Settings.  After the Settings key
 	add values as you see fit.
 	
 	Here is an example, as used in the FuncGuard module.
 	
 	PrivateData = @{
 		"LocalUser" = @{
 			"Settings"=@{
 				"ActiveConfiguration" = "PSUnit"
 				"Configurations"=@{
 					"PSUnit" =@{
 						"Name"="PSUnit";
 						"Path"=$(join-path -Path $Env:Temp -ChildPath "FuncGuard.PSUnit");
 						"Prefix"="PSUNIT"
 					}
 				}
 			}
 		}
 	}
 	
 	After loading the module a global variable will be available with a name
 	convention of $($ModuleName)Settings and the values can be accessed using
 	normal hash retrieval.
 	
 	$config = $FuncGuardSettings["Configurations"].$($FuncGuardSettings["ActiveConfiguration"])
 	$Path = $config.Path
 
 #>
 
 Set-StrictMode -Version 2
 
 $Module = $ExecutionContext.SessionState.Module
 if (! $Module) {
 	throw ( New-Object System.InvalidOperationException `
 		"An active module was not found!  LoadModuleConfig must be run from a module manifest's NestedModules field.")
 }
 $ModuleName = $Module.Name.Replace(".config",$null)
 
 $AppData = [System.Environment]::GetFolderPath([System.Environment+SpecialFolder]::LocalApplicationData)
 $ModuleConfigPath = join-path -Path $AppData `
 	-ChildPath "WindowsPowerShell\Modules\$ModuleName\$ModuleName.config.xml"
 
 if (! (Test-Path $ModuleConfigPath -PathType Leaf) ) {
 	$configurations = $Module.PrivateData.LocalUser.Settings
 	$configurations.add("InitializeConfigurations", $true)
 	Export-Clixml -InputObject $configurations -Path $ModuleConfigPath
 }
 
 $configurations = Import-Clixml -Path $ModuleConfigPath
 
 Add-Member -InputObject $configurations -Name ModuleName -MemberType NoteProperty -force `
 	-Value $ModuleName
 
 Add-Member -InputObject $configurations -Name Export -MemberType ScriptMethod -force `
 	-Value {
 		$AppData = [System.Environment]::GetFolderPath([System.Environment+SpecialFolder]::LocalApplicationData)
 		$ModuleConfigPath = join-path -Path $AppData `
 			-ChildPath "WindowsPowerShell\Modules\$ModuleName\$($this.ModuleName).config.xml"
 		Export-Clixml -InputObject $this -Path $ModuleConfigPath
 	}
 
 Set-Variable -Name "$($ModuleName)Settings" -Value $($configurations) -Scope Global `
 	-Description "$($ModuleName) module configuration settings."
`

