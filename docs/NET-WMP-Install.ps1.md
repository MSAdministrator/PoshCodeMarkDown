---
Author: zerfam
Publisher: 
Copyright: 
Email: 
Version: 4.5
Encoding: ascii
License: cc0
PoshCode ID: 5622
Published Date: 2014-12-01t17
Archived Date: 2014-12-13t19
---

# .net/wmp install - 

## Description

just a simple script to tie to a gpo to install .net 4.5 and wmf 4.0

## Comments



## Usage



## TODO



## function

``

## Code

`#
 #
 $DeploymentDir = '\\Server\Deployment\DotNET4.6_WMF4'
 
 function message
 {
 	param 
 	(
 		[parameter(Mandatory=$true)]
 		[string]$ErrorMessage
 	)
 	
 	$logFile = '\\Server\Logs\dotnet_deploy.log'
 	
 	$msg = "$env:ComputerName - "
 	$msg += Get-Date -Format "yyyy-MM-dd h:mm tt - "
 	$msg += $ErrorMessage
 	
 	Add-Content $logFile "$StartTime - $msg "
 	
 	return $msg
 }
 
 $reg = Get-Item 'HKLM:\SOFTWARE\Microsoft\.NETFramework\v4.0.30319\SKUs\.NETFramework,Version=v4.5' -ErrorAction SilentlyContinue
 if (-not $reg)
 {
 	
 	message "Installing .NET 4.5."
 	try
 	{
 		Start-Process -Wait -FilePath "$DeploymentDir\NDP451-KB2858728-x86-x64-AllOS-ENU.exe" -ArgumentList '/CEIPconsent /norestart /q'
 	}
 	catch [System.Exception]
 	{
 		message "Failed to run .NET installer: $_"
 	}
 }
 else
 {
 	message ".NET 4.5 already installed."
 }
 
 if($PSVersionTable.PSVersion.Major -ne '4')
 {
 	$OS = (GWMI Win32_Processor).AddressWidth
 
 	if($OS -eq "32")
 	{
 		message "Installing WMF4 32-bit."
 		try
 		{
 			Start-Process -Wait -FilePath 'wusa.exe' -ArgumentList "$DeploymentDir\Windows6.1-KB2819745-x86-MultiPkg.msu /quiet /norestart"
 		}
 		catch [System.Exception]
 		{
 			message "Failed to run .NET installer: $_"
 		}
 	}
 	elseif($OS -eq "64")
 	{
 		message "Installing WMF4 64-bit."
 		try
 		{
 			Start-Process -Wait -FilePath 'wusa.exe' -ArgumentList "$DeploymentDir\Windows6.1-KB2819745-x64-MultiPkg.msu /quiet /norestart"
 		}
 		catch [System.Exception]
 		{
 			message "Failed to run .NET installer: $_"
 		}
 	}
 }
 else
 {
 	message "WMF4 already installed."
 }
`

