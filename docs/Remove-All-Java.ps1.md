---
Author: john delise
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6079
Published Date: 2017-11-05t21
Archived Date: 2017-05-30t18
---

# remove all java - 

## Description

this script will remove all versions of java run-time, both 32-bit and 64-bit if the uninstall files in the bin folder of each version are present. if they are not then a version(s) could be left behind.	we use this when a users system has it’s java run-time.  when you have multiple versions of java jre installed in static mode and need to clear them up to level set and reinstall the new versions.  no system restarts or log offs required.

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 
 #---------------------------------------------------------------------------------------------------------------------------
 
 @echo off
 CLS
 SETLOCAL
 
 SET INST=%~dp0
 call "%INST%Stop_Tasks.cmd"
 "Powershell.exe" -NoLogo -NoProfile -Windowstyle Normal -Executionpolicy unrestricted  -file  "%INST%Remove_All_Java.ps1"
 "Powershell.exe" -NoLogo -NoProfile -Windowstyle Normal -Executionpolicy unrestricted  -file  "%INST%Remove-RegKey.ps1"
 
 
 
 ENDLOCAL
 GOTO :EOF
 #---------------------------------------------------------------------------------------------------------------------------
 
 
 
 Clear-Host
 
 
 $_0 = Get-Location
 
 
 set-location 'HKLM:\software\microsoft\windows\currentversion\uninstall'
 $_Uninstall1 = get-childitem -path (Get-location) 
 
 
 set-location 'HKLM:\SOFTWARE\Wow6432Node\Microsoft\Windows\CurrentVersion\Uninstall'
 $_Uninstall2 = get-childitem -path (Get-location)
 
 $_uninstall3 = $_Uninstall1 + $_Uninstall2
 
 Set-location $_0
 
 
 Foreach($_Key in $_uninstall3){
 	
 	$_Key_Property_Path = $_Key.Name.Replace('HKEY_LOCAL_MACHINE\','HKLM:\')
 	
 
 	If($_Key.ValueCount -eq 0){continue}
 	
 	#$_Key.ValueCount
 	$_Reg_Name 				= $_Key.PSDrive.Name
 	$_Reg_ChildName			= $_Key.PSChildName
 	$_Reg_CurrentLocation	= $_Key.PSDrive.CurrentLocation
 	$_Reg_Value_ToQuery		= $_Key_Property_Path 	
 	$_Reg_PropertiesArray	= get-itemproperty -path    $_Reg_Value_ToQuery
 	
 	
 	If($_Reg_PropertiesArray.DisplayName -eq $NULL) {Continue}
 	If($_Reg_PropertiesArray.DisplayName -eq 'Java Auto Updater') {Continue}
 	
 	If($_Reg_PropertiesArray.DisplayName.Contains('Java')) {
 			$_DisplayNameStr = $_Reg_PropertiesArray.DisplayName.ToString()
 			Write-host "Removing $_DisplayNameStr"
 			
 			$_InstallPath 		= $_Reg_PropertiesArray.InstallLocation.ToString()
             If($_InstallPath -eq ''){
             
                  $_VersionArray =  $_DisplayNameStr.Split(' ')
                  $_Version = '1.'+ $_VersionArray[1] + '.0_' + $_VersionArray[3]
             
                 If($Env:PROCESSOR_ARCHITECTURE -eq 'x86' )   {$_installPath = ('C:\Program Files\Java\jre' + $_Version + '\')}
                 If($Env:PROCESSOR_ARCHITECTURE -eq 'AMD64' ) {$_installPath = ('C:\Program Files (x86)\Java\jre' + $_Version + '\')}
 
 
             
             }
 			$_InstallPathFQN 	= $_installPath + 'lib\Deployment.config'
 			$_DConfig_Present 	= Test-Path -path $_InstallPathFQN
 			$_DConfig_Present
 			
 			IF($_DConfig_Present){
 			
 				If($_DConfig_Present -ne '') {
 				
 					Remove-Item -Path $_InstallPathFQN -Force
 					
 				}
 			
 			
 			}
 			
 			$_exe 	= 	'Msiexec.exe'
 			$_args 	= 	'/uninstall',
       	 				"$_Reg_ChildName",
 						'/passive',
 						'/norestart'
 			
 			& $_exe $_args		| OUT-HOST	
 
 			Write-Host ''
 		}
 	}
  }
 
 $_InstallPathFQN 	= 'C:\Program Files\Java'
 $_DConfig_Present 	= Test-Path -path $_InstallPathFQN
 $_DConfig_Present
 			
 IF($_DConfig_Present){
 			
 	If($_DConfig_Present -ne '') {
 				
 		Remove-Item -Path $_InstallPathFQN -Force -Recurse
 					
 	}
 			
 			
 }
 
 $_InstallPathFQN 	= 'C:\Program Files (x86)\Java'
 $_DConfig_Present 	= Test-Path -path $_InstallPathFQN
 $_DConfig_Present
 			
 IF($_DConfig_Present){
 			
 	If($_DConfig_Present -ne '') {
 				
 		Remove-Item -Path $_InstallPathFQN -Force -Recurse
 					
 	}
 			
 			
 }
 
 #-----------------------------------------------------------------------------------------------------------------------------------------------------------
 
 <#
 .SYNOPSIS
    Remove a Regkey
 .DESCRIPTION
    Removes one Regkey and all it's subkeys and values
 .PARAMETER <paramName>
    None
 .EXAMPLE
    PowerShell.exe -noprofile -executionpolicy bypass -Nologo  -noexit -File "%INST%Remove-RegKey.ps1"
    Where %INST% is the batch file variable SET INST=%~dp0, set this before running if from a batch file.
 #>
 Clear-host
 $_REG_DRIVE 	= "HKLM:"
 $_REG_KEY 		= '\SOFTWARE\Classes\CLSID\`{4299124F-F2C3-41b4-9C73-9236B2AD0E8F`}'
 $_Path_Exists	= Test-Path 'HKLM:\SOFTWARE\Classes\CLSID\{4299124F-F2C3-41b4-9C73-9236B2AD0E8F}' 
 IF($_Path_Exists){
 	Write-Host "$_REG_KEY found`n`n" -ForegroundColor BLACK -BackgroundColor YELLOW
 	$_Reg_Result 	= Remove-Item 'HKLM:\SOFTWARE\Classes\CLSID\{4299124F-F2C3-41b4-9C73-9236B2AD0E8F}' -Recurse
 	
 	$_Path_Exists	= Test-Path 'HKLM:\SOFTWARE\Classes\CLSID\{4299124F-F2C3-41b4-9C73-9236B2AD0E8F}' 
 	IF($_Path_Exists){
 	
 		Write-Host "$_REG_KEY found, DELETION FAILED.`n`n" -ForegroundColor RED -BackgroundColor yellow
 	
 	}Else{
 		Write-Host "The Reg Key $_REG_KEY was found, it has been deleted as requested.`n" -ForegroundColor Green -BackgroundColor BLACK
 	}
 }Else{
 Write-Host "The Reg Key $_REG_KEY was not found, no deletion needed.`n`n" -ForegroundColor Green -BackgroundColor BLACK
 }
`

