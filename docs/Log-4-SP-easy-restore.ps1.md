---
Author: cglessner
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 1799
Published Date: 
Archived Date: 2010-04-24t23
---

# log 4 sp easy restore - 

## Description

log sharepoint content deletions in all sites fora specified web application. motivation was to determine content deletion dates to easily find the right backup for a selective restore.

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 
 
 
 $targetWebAppUrl = "http://localhost:100"
 
 $logWebUrl = "http://localhost:100"
 
 $logListTitle ="Audit"
 
 
 $lastRunPropName = "_iLSP_lastAuditTimestamp"
 
 [void][System.Reflection.Assembly]::LoadWithPartialName('Microsoft.SharePoint')
 
 $logSite = New-Object Microsoft.SharePoint.SPSite($logWebUrl)
 $logWeb = $logSite.OpenWeb()
 $logList = $logWeb.Lists[$logListTitle]
 
 $targetWebApp = [Microsoft.SharePoint.Administration.SPWebApplication]::Lookup($(New-Object Uri($targetWebAppUrl)))
 
 if($targetWebApp.Properties.ContainsKey($lastRunPropName) -eq $false)
 {
 	$targetWebApp.Properties[$lastRunPropName] = [DateTime]::MinValue
 	$targetWebApp.Update()
 }
 
 $lastRun = [DateTime]::Parse($targetWebApp.Properties[$lastRunPropName].ToString())
 $newRun = [DateTime]::Now.ToUniversalTime()
 
 foreach($site in $targetWebApp.Sites)
 {	
 	$query = New-Object Microsoft.SharePoint.SPAuditQuery($site)
 	$query.AddEventRestriction([Microsoft.SharePoint.SPAuditEventType]::Delete)
 	$query.SetRangeStart($lastRun)
 	$query.SetRangeEnd($newRun.AddDays(1))
 	
 	$result = $site.Audit.GetEntries($query)
 	
 	if($result.Count -gt 0)
 	{
 		foreach($entry in $result)
 		{
 			$eventUrl = $site.Url + "/" + $entry.DocLocation
 			
 			$item = $logList.Items.Add()
 			$item[[Microsoft.SharePoint.SPBuiltInFieldId]::Title] = $entry.DocLocation
 			$item["Event Url"] = $eventUrl
 			$item["Item Type"] = $entry.ItemType
 			$item["Event"] = $entry.Event
 			$item["Occurred"] = $entry.Occurred.ToLocalTime()
 			$item["User"] = $logWeb.SiteUsers.GetByID($entry.UserId).LoginName
 			$item.Update()
 			
 		}
 	}
 	
 	$site.Dispose()
 }
 
 $targetWebApp.Properties[$lastRunPropName] = $newRun
 $targetWebApp.Update()
 
 $logWeb.Dispose()
 $logSite.Dispose()
`

