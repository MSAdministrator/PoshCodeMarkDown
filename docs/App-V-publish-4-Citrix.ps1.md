---
Author: thegeek
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6332
Published Date: 2016-05-02t03
Archived Date: 2016-07-04t13
---

# app-v publish 4 citrix - 

## Description

generic script for citrix pvs and app-v to publish apps without the need for a management and database server for app-v.  generic script doesn’t need to be updated for every change or app so the image wont have to be opened and sealed every time.  the published app in the xenapp console passes the necessary information for each new published app, making it easy to add apps on the fly and not have to modify the pvs image.  this script should be not necessary if you have xenapp 7.8 or higher.

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 ###################################################################
 ##
 ##
 ###################################################################
 
 
 param(
 	[string]$appName,
 	[string]$exeName
 )
 
 Write-Host "`n`n                        Your application is loading..."
 
 Import-Module AppvClient
 $package = Get-AppVClientPackage -Name $appName
 
 If ( !$package ) {
 	Try {
 		$package = Add-AppVClientPackage \\PathToApp-VPackageStorage\$appName\$appName.appv
 		Publish-AppVClientPackage $appName -Global | Out-Null
 		$didComplete = $true
 	} Catch {
 		$errorMessage = "Error Item: $_.Exception.ItemName `n`nError Message: $_.Exception.Message"
 		$didComplete = $false
 		$wshell = New-Object -ComObject Wscript.Shell
 		$wshell.Popup($errorMessage,0,"Error loading application",0x0)
 	}
 } else {
 	$didComplete = $true
 }
 
 $windowTitleCount = (get-process | where {$_.MainWindowTitle}).Count
 
 If ($didComplete) { 
 	$cmdExpand = "cmd /c echo $( (Get-AppVClientConfiguration | where {$_.Name -eq 'PackageInstallationRoot'}).Value )"
 	$packageRoot = Invoke-Expression $cmdExpand
 	$packagePath = "$packageRoot\$($package.PackageId)\$($package.VersionId)\Root\VFS"
 	[XML]$packageConfig = Get-Content "PathToApp-VPackageStorage\$appName\$($appName)_DeploymentConfig.xml"
 	$packageApps = $packageConfig.DeploymentConfiguration.UserConfiguration.Applications.Application.Id
 	ForEach ($app in $packageApps) {
 		If ($app -match $exeName) {
 			$appPath = $app
 			break
 		}
 	}
 	$fullPath = "$packagePath\$appPath"
 	Start $fullPath
 	
 	$currentTitleCount = (get-process | where {$_.MainWindowTitle}).Count
 	While ($windowTitleCount -ge $currentTitleCount) {
 		$currentTitleCount = (get-process | where {$_.MainWindowTitle}).Count
 	}
 }
`

