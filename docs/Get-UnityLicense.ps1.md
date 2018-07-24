---
Author: robbie foust (rfoust@duke.edu)
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 750
Published Date: 2011-12-26t16
Archived Date: 2011-05-02t01
---

# get-unitylicense - 

## Description

this function connects via http to a cisco unity server and returns license information as a pscustomobject.

## Comments



## Usage

get-unitylicense <server>

## TODO



## function

``

## Code

`#
 #
 #
 #
 #
 #
 
 function global:get-unitylicense ([string]$server = $(throw "Please provide a server name!"))
 	{
 
 	$webContent = new-object net.webclient
 	$page = $webContent.DownloadString("http://$server/avxml/effectivelicense.asp")
 
 	$page = $page -replace "^.`n"
 	$license = [xml]$page
 
 	new-object psobject | add-member -memberType NoteProperty -name LicLanguagesMax -value $license.AvXmlLicData.Licenses.LicLanguagesMax -passthru |
 		add-member -memberType NoteProperty -name LicMaxMsgRecLenIsLicensed -value $license.AvXmlLicData.Licenses.LicMaxMsgRecLenIsLicensed -passthru |
 		add-member -memberType NoteProperty -name LicPoolingIsEnabled -value $license.AvXmlLicData.Licenses.LicPoolingIsEnabled -passthru |
 		add-member -memberType NoteProperty -name LicSubscribersMax -value $license.AvXmlLicData.Licenses.LicSubscribersMax -passthru |
 		add-member -memberType NoteProperty -name LicUMSubscribersMax -value $license.AvXmlLicData.Licenses.LicUMSubscribersMax -passthru |
 		add-member -memberType NoteProperty -name LicVMISubscribersMax -value $license.AvXmlLicData.Licenses.LicVMISubscribersMax -passthru |
 		add-member -memberType NoteProperty -name LicVoicePortsMax -value $license.AvXmlLicData.Licenses.LicVoicePortsMax -passthru |
 		add-member -memberType NoteProperty -name AvLicUtilizationSecondaryServer -value $license.AvXmlLicData.Utilization.AvLicUtilizationSecondaryServer -passthru |
 		add-member -memberType NoteProperty -name AvLicUtilizationSubscribers -value $license.AvXmlLicData.Utilization.AvLicUtilizationSubscribers -passthru |
 		add-member -memberType NoteProperty -name AvLicUtilizationVMISubscribers -value $license.AvXmlLicData.Utilization.AvLicUtilizationVMISubscribers -passthru
 	}
`

