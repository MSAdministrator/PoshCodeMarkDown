---
Author: jgrote
Publisher: 
Copyright: 
Email: 
Version: 4.5
Encoding: ascii
License: cc0
PoshCode ID: 2089
Published Date: 2013-08-18t22
Archived Date: 2013-05-10t08
---

# start-appvtestmode - 

## Description

this script allows testing of newly sequenced app-v apps without having to specify the file

## Comments



## Usage



## TODO



## script

`show-messagebox`

## Code

`#
 #
 
 function Show-MessageBox ($title, $msg) {	
 	[System.Reflection.Assembly]::LoadWithPartialName("System.Windows.Forms") | Out-Null
 	[Windows.Forms.MessageBox]::Show($msg, $title, [Windows.Forms.MessageBoxButtons]::OK, [System.Windows.Forms.MessageBoxIcon]::Warning, [System.Windows.Forms.MessageBoxDefaultButton]::Button1, [System.Windows.Forms.MessageBoxOptions]::DefaultDesktopOnly) | Out-Null	
 }
 
 function Show-InformationBox ([string]$Title,[string]$Message) {
 	[System.Reflection.Assembly]::LoadWithPartialName("System.Windows.Forms") | Out-Null
 	[Windows.Forms.MessageBox]::Show($Message, $Title, [Windows.Forms.MessageBoxButtons]::OK, [System.Windows.Forms.MessageBoxIcon]::Information, [System.Windows.Forms.MessageBoxDefaultButton]::Button1, [System.Windows.Forms.MessageBoxOptions]::DefaultDesktopOnly) | Out-Null	
 }
 
 $Repository = "\\server\share\AppVRepository\2-TestPackages"
 
 $AppVRegRoot = "HKLM:\SOFTWARE\Microsoft\SoftGrid\4.5\Client"
 
 if ((Get-WMIObject win32_operatingsystem).OSArchitecture -match "64-bit") {
 	$AppVRegRoot = "HKLM:\SOFTWARE\WOW6432Node\Microsoft\SoftGrid\4.5\Client"
 }
 
 $transEnterAppVTestMode = start-transaction
 
 set-location $AppVRegRoot
 
 $ASROriginalSetting = (get-itemproperty Configuration).ApplicationSourceRoot
 $AIFSOriginalSetting = (get-itemproperty Configuration).AllowIndependentFileStreaming
 
 set-itemproperty Configuration -name "ApplicationSourceRoot" -value $repository
 set-itemproperty Configuration -name "IconSourceRoot" -value $repository
 
 sftmime DELETE obj:app /global
 
 set-location C:
 pwd
 set-location $repository
 pwd
 $i=1
 get-childitem $repository | where {$_.Name -match "manifest.xml"} | foreach {
 	echo $_.Name
 	sftmime ADD PACKAGE:TestApp$i /manifest $_.Name /global
 	$i++
 }
 
 Show-InformationBox "App-V Testing Mode is active" "Apps will be sourced from $Repository. 
 
 NOTE: File Associations (launching programs by double-clicking their files like .docx) will not be testable due to SCCM vAppLauncher bug
 
 Click OK to return to normal mode"
 
 set-location $AppVRegRoot
 pwd
 
 set-itemproperty Configuration -name "ApplicationSourceRoot" -value ""
 set-itemproperty Configuration -name "IconSourceRoot" -value ""
 
 sftmime DELETE obj:app /global
`

