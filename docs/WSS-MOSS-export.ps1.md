---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 137
Published Date: 
Archived Date: 2008-09-08t00
---

# wss/moss export - 

## Description

how to export sharepoint with help of powershell

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 #
 
 param ( [string] $sitename = "http://localhost:80", [string] $relweburl = "/wiki")
 
 [void] [System.Reflection.Assembly]::Load("Microsoft.SharePoint, Version=12.0.0.0, Culture=neutral, PublicKeyToken=71e9bce111e9429c") 
 
 
 $spsite = new-object Microsoft.SharePoint.SPSite($SiteName)
 $webpart = $spsite.openweb($relweburl, $false)
 
 
 if (!$webpart -is [Microsoft.SharePoint.SPWeb]) {
   Write-Host "Wrong type to export!" -foregroundcolor red
   exit
 }
 
 
 [System.Guid] $webguid = $webpart.ID
 
 
 [Microsoft.SharePoint.Deployment.SPExportObject] $exportObject = new-object Microsoft.SharePoint.Deployment.SPExportObject
 $exportObject.Id = $webguid   
 $exportObject.Type = [Microsoft.SharePoint.Deployment.SPDeploymentObjectType]::Web
 $exportObject.ExcludeChildren = $FALSE
 $exportObject.IncludeDescendants = [Microsoft.SharePoint.Deployment.SPIncludeDescendants]::All
 
 [Microsoft.SharePoint.Deployment.SPExportSettings] $expSettings = new-object Microsoft.SharePoint.Deployment.SPExportSettings
 $expSettings.SiteUrl = $sitename
 $expSettings.ExportObjects.Add($exportObject)
 $expSettings.FileCompression = $TRUE
 $expSettings.OverwriteExistingDataFile = $TRUE
 $expSettings.BaseFileName = "export.cab"
 $expSettings.FileLocation = "."
 $expSettings.LogFilePath = "./export.log"
 $expSettings.IncludeVersions = [Microsoft.SharePoint.Deployment.SPIncludeVersions]::LastMajor
 $expSettings.ExcludeDependencies = $FALSE
 $expSettings.Validate()
 $expSettings
 
 [Microsoft.SharePoint.Deployment.SPExport] $export = new-object Microsoft.SharePoint.Deployment.SPExport($expSettings)
 $export.Run()
 
 Write-Host "Done!"
 
 trap [Exception] { 
       write-host $("`tTRAPPED: " + $_.Exception.GetType().FullName)
       write-host $("`tTRAPPED: " + $_.Exception.Message) 
       break
 }
`

