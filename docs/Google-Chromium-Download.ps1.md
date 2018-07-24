---
Author: mjwj1
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2959
Published Date: 2013-09-18t09
Archived Date: 2013-09-28t00
---

# google chromium download - 

## Description

download and install google chromium if there is a newer version available.

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 <#
     .Synopsis
         Download Google Chromium if there is a later build.
     .Description
         Download Google Chromium if there is a later build.        
         http://poshcode.org/2422
 #>
 
 Set-StrictMode -Version Latest
 Import-Module bitstransfer
 
 $VerbosePreference = "Continue"
 #$VerbosePreference = "SilentlyContinue"
 
 $versionFile = "$Env:temp\latestChromiumVersion.txt"
 $installedChromiumVersion = 0
 
 trap [Exception] { 
       write-host
       write-error $("TRAPPED: " + $_.Exception.GetType().FullName); 
       write-error $("TRAPPED: " + $_.Exception.Message); 
       [string]($installedChromiumVersion) > $versionFile
       exit; 
 }
 
 
 if (Test-Path $versionFile)
 { $installedChromiumVersion = [int32] (cat $versionFile) }
 
 #$latestChromiumBuildURL ="http://build.chromium.org/f/chromium/snapshots/chromium-rel-xp"
 #$latestChromiumBuildURL ="http://build.chromium.org/f/chromium/snapshots/Win"
 $latestChromiumBuildURL ="http://commondatastorage.googleapis.com/chromium-browser-snapshots/Win"
 Start-BitsTransfer "$latestChromiumBuildURL/LAST_CHANGE" $versionFile
 $latestChromiumVersion = [int32] (cat $versionFile)
 
 if ($installedChromiumVersion -eq $latestChromiumVersion)
 { 
     Write-Verbose "Exiting... Version $installedChromiumVersion is the latest. (from $latestChromiumBuildURL/LAST_CHANGE)"
     return
 }
 
 $installerAppName = "mini_installer"
 $installer = "$Env:temp\$installerAppName.exe"
 Write-Verbose "Initiating download of new version $latestChromiumVersion"
 Start-BitsTransfer "$latestChromiumBuildURL/$latestChromiumVersion/mini_installer.exe" $installer
 Write-Verbose "Installing new version of Chromium"
 Invoke-Item $installer
 $installerRunning = 1
 while (!($installerRunning -eq $null))
 { 
     sleep 5
     $installerRunning = ( Get-Process | ? {$_.ProcessName -match "$installerAppName"} )
 }
 
 Write-Verbose "New Chromium Installed! Please restart the Chromium browser"
`

