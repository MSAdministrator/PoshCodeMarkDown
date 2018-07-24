---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 3.5
Encoding: ascii
License: cc0
PoshCode ID: 1050
Published Date: 
Archived Date: 2010-01-25t15
---

# work-in-progressspdeploy - 

## Description

work-in-progress. this script is meant to be run from a “scripts/” subdirectory as part of a larger build structure. it won’t run by itself, but maybe you’ll find the sharepoint deployment bits useful.

## Comments



## Usage



## TODO



## function

`run-msbuild`

## Code

`#
 #
 
 
 [System.Reflection.Assembly]::Load('Microsoft.Build.Utilities.v3.5, Version=3.5.0.0, Culture=neutral, PublicKeyToken=b03f5f7f11d50a3a') | out-null
 [System.Reflection.Assembly]::Load('Microsoft.SharePoint, Version=12.0.0.0, Culture=neutral, PublicKeyToken=71e9bce111e9429c') | out-null
 $msbuild = [Microsoft.Build.Utilities.ToolLocationHelper]::GetPathToDotNetFrameworkFile("msbuild.exe", "VersionLatest")
 
 $global:basepath = (resolve-path ..).path
 $stsadm = Join-Path ([Microsoft.SharePoint.Utilities.SPUtility]::GetGenericSetupPath("BIN")) "stsadm.exe"
 
 
 function Run-MSBuild($msBuildArgs)
 {
 	& $msbuild $msBuildArgs
 }
 
 function Get-FirstDirectoryUnderneathSrc
 {
 	dir (Get-FullPath "src") | where { $_.PSIsContainer -eq $true } | select -first 1
 }
 
 function Get-FullPath($subdirectory)
 {
 	return join-path -path $basepath -childPath $subdirectory
 }
 
 $wspbuilder = Get-FullPath("tools\WSPBuilder.exe")
 function Run-WspBuilder($rootDirectory)
 {
 	pushd
 	cd $rootDirectory
 	& $WSPBuilder -BuildWSP true -OutputPath (Get-FullPath 'deployment')
 	popd
 }
 
 function Clean-Path($dir)
 {
 	del $dir -recurse -force -erroraction SilentlyContinue
 	mkdir $dir -erroraction SilentlyContinue | out-null
 }
 
 function Create-DeploymentBatchFile($filename, $featureName, $solutionName, $url)
 {
 	$contents = @"
 
 ECHO OFF
 SET STSADM="%PROGRAMFILES%\Common Files\Microsoft Shared\web server extensions\12\BIN\stsadm.exe"
 %STSADM% -o deactivatefeature -name $featureName -url $url
 %STSADM% -o retractsolution -allcontenturls -immediate -name $solutionName
 %STSADM% -o execadmsvcjobs
 %STSADM% -o deletesolution -name $solutionName -override
 %STSADM% -o addsolution -filename $solutionName
 %STSADM% -o deploysolution -allcontenturls -immediate -allowgacdeployment -name $solutionName
 %STSADM% -o execadmsvcjobs
 REM second call to execadmsvcjobs allows for a little more delay. Shouldn't be necessary, but is.
 %STSADM% -o execadmsvcjobs
 %STSADM% -o activatefeature -url $url -name $featureName
 "@
 
 	Out-File -inputObject $contents -filePath $filename -encoding ASCII
 }
 
 function Do-Deployment($featureName, $solutionName, $rootDirectory)
 {
 	echo $featureName, $solutionName, $rootDirectory
 	if (-not (Is-Installed $solutionName)) {
 		Install-Solution -solutionName $solutionName -filename (join-path $rootDirectory $solutionName)	
 	}
 	
 	Exec-AdmSvcJobs
 	
 	if (-not (Is-Deployed $solutionName)) {
 		Deploy-Solution -solutionName $solutionName -featureName $featureName -filename (join-path $rootDirectory $solutionName)
 	} else {
 		Upgrade-Solution -solutionName $solutionName -featureName $featureName -filename (join-path $rootDirectory $solutionName)
 	}
 }
 
 function Is-Installed($solutionName)
 {
 	return [Microsoft.SharePoint.Administration.SPFarm]::Local.Solutions[$solutionName] -ne $null
 }
 
 function Is-Deployed($solutionName)
 {
 	$solution = [Microsoft.SharePoint.Administration.SPFarm]::Local.Solutions[$solutionName]
 	if ($solution -eq $null) { return false; }
 	
 	return $solution.Deployed
 }
 
 function Install-Solution($solutionName, $filename)
 {
 	[Microsoft.SharePoint.Administration.SPFarm]::Local.Solutions.psbase.Add($filename) | out-null
 }
 
 function Deploy-Solution($featureName, $solutionName, $filename)
 {
 	$dateToGuaranteeInstantDeployment = [datetime]::Now.AddDays(-2)
 	$webApplications = [Microsoft.SharePoint.Administration.SPWebService]::ContentService.WebApplications | % { $_ }
 	$webApplicationsCollection = new-object Microsoft.SharePoint.Administration.SPWebApplication[] -arg ($webApplications.Count)
 	0..($webApplications.Count-1) | % { $webApplicationsCollection[$_] = $webApplications[$_] }
 	
 	[Microsoft.SharePoint.Administration.SPFarm]::Local.Solutions[$solutionName].Deploy($dateToGuaranteeInstantDeployment, $true, $webApplicationsCollection, $false)
 }
 
 function Upgrade-Solution($featureName, $solutionName, $filename)
 {
 	[Microsoft.SharePoint.Administration.SPFarm]::Local.Solutions[$solutionName].Upgrade($filename)
 }
 
 function Exec-AdmSvcJobs
 {
 	& $stsadm -o execadmsvcjobs
 	sleep -seconds 2
 }
`

