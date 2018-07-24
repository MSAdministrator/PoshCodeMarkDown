---
Author: jayneticmuffin
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5670
Published Date: 2015-01-07t14
Archived Date: 2015-01-31t21
---

# automatic windows update - 

## Description

the script will launch windows update on your machine, and display the patches as they are being installed.  once installed, a result code will be displayed as well. if a reboot is needed, the script will take care of that automatically.

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 Write-Host "  Checking for available updates... Please wait!" -ForegroundColor 'Yellow'
 $UpdateSession = New-Object -ComObject Microsoft.Update.Session
 $SearchResult = $UpdateSession.CreateUpdateSearcher().Search($criteria).Updates | ? {$_.Title -notmatch $skipUpdates}
 $SearchResult = $SearchResult | Sort-Object LastDeploymentChangeTime -Descending
 $totalUpdates = $SearchResult.Count
 
 ForEach ($Update in $SearchResult) {
 	If ($totalUpdates -eq $null) { $totalUpdates = 1}
 	$UpdatesCollection = New-Object -ComObject Microsoft.Update.UpdateColl
 	if ($Update.EulaAccepted -eq 0) { $Update.AcceptEula() }
 	$null = $UpdatesCollection.Add($Update)
 	
 	$fileSize = "{0:N0} MB" -f ($($Update.MaxDownloadSize) / 1MB)
 	Write-Host "  Downloading ($($counter.ToString("00"))/$($totalUpdates.ToString("00"))) - $fileSize - " -ForegroundColor 'Yellow' -NoNewline
 	$counter++
 	Write-Host "$($Update.Title)" -ForegroundColor 'White'
 	$UpdatesDownloader = $UpdateSession.CreateUpdateDownloader()
 	$UpdatesDownloader.Updates = $UpdatesCollection
 	$UpdatesDownloader.Priority = 3
 
 	Write-Host "`t- Download " -NoNewline -ForegroundColor 'White'
 	$DownloadResult = $UpdatesDownloader.Download()
 	Switch ($DownloadResult.ResultCode) {
 		0   { Write-Host "Not Started" -ForegroundColor 'Black' -BackgroundColor 'Yellow' }
 		1   { Write-Host "In Progress" -ForegroundColor 'Black' -BackgroundColor 'Yellow' }
 		2   { Write-Host "Succeeded" -ForegroundColor 'Green' }
 		3   { Write-Host "Succeeded (with Errors)" -ForegroundColor 'Black' -BackgroundColor 'Yellow' }
 		4   { Write-Host "Failed" -ForegroundColor 'White' -BackgroundColor 'Red' }
 		5   { Write-Host "Aborted" -ForegroundColor 'White' -BackgroundColor 'Red' }
 	}
 
 	$UpdatesInstaller = $UpdateSession.CreateUpdateInstaller()
 	$UpdatesInstaller.Updates = $UpdatesCollection
 
 	Write-Host "`t- Install  " -NoNewline -ForegroundColor 'White'
 	$InstallResult = $UpdatesInstaller.Install()
 	Switch ($installResult.ResultCode) {
 		0   { Write-Host "Not Started" -ForegroundColor 'Black' -BackgroundColor 'Yellow' }
 		1   { Write-Host "In Progress" -ForegroundColor 'Black' -BackgroundColor 'Yellow' }
 		2   { Write-Host "Succeeded" -ForegroundColor 'Green' }
 		3   { Write-Host "Succeeded (with Errors)" -ForegroundColor 'Black' -BackgroundColor 'Yellow' }
 		4   { Write-Host "Failed" -ForegroundColor 'White' -BackgroundColor 'Red' }
 		5   { Write-Host "Aborted" -ForegroundColor 'White' -BackgroundColor 'Red' }
 	}
 	If ($installResult.rebootRequired -eq 'True') { $needsReboot = $true }
 }
`

