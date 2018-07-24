---
Author: nathan linley
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 3214
Published Date: 2012-02-08t19
Archived Date: 2012-04-14t08
---

# compare-dellupdates - 

## Description

this script will compare the bios/firmware/driver/omsa versions of a remote server against the dell suu update package.  to use it, get a copy of the latest dell suu.  inside the suu file structure, look for catalog.xml in the repository.  this file will be read to determine the latest versions, and compare it to the currently installed versions on the remote machine (reporting any differences).  the output can be used to easily push updates as well, since the update file name is returned in the results.

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 
 param(
 	[parameter(mandatory=$true)][ValidateScript({test-path $_ -pathtype 'leaf'})][string]$catalogpath,
 	[parameter(mandatory=$true,ValueFromPipeline=$true)][string]$server
 )
 
 function changedatacase([string]$str) {
 	if ($str -match "`=`"") {
 		$myparts = $str.split("=")
 		$result = $myparts[0] + "=" + $myparts[1].toupper()
 		return $result
 	} else { return $str}
 }
 
 $catalog = [xml](get-Content $catalogpath)
 $oscodeid = &{
 	$caption = (Get-WmiObject win32_operatingsystem -ComputerName $server).caption
 	if ($caption -match "2003") {
 		if ($caption -match "x64") { "WX64E" } else { "WNET2"}
 	} elseif ($caption -match "2008 R2") { 
 		"W8R2" 
 	} elseif ($caption -match "2008" ) {
 			if ($caption -match "x64") { 
 				"WSSP2" 
 			} else {
 				"LHS86"
 			}	
 	}
 }
 write-debug $oscodeid
 
 $systemID = (Get-WmiObject -Namespace "root\cimv2\dell" -query "Select Systemid from Dell_CMInventory" -ComputerName $server).systemid
 $model = (Get-WmiObject -Namespace "root\cimv2\dell" -query "select Model from Dell_chassis" -ComputerName $server).Model
 
 $devices = Get-WmiObject -Namespace "root\cimv2\dell" -Class dell_cmdeviceapplication -ComputerName $server
 foreach ($dev in $devices) {
 	$xpathstr = $parts = $version = ""
 	if ($dev.Dependent -match "(version=`")([A-Z\d.-]+)`"") { $version = $matches[2]	} else { $version = "unknown" }
 	$parts = $dev.Antecedent.split(",")
 	for ($i = 2; $i -lt 6; $i++) {
 		$parts[$i] = &changedatacase $parts[$i]
 	}
 	$depparts = $dev.dependent.split(",")
 	$componentType = $depparts[0].substring($depparts[0].indexof('"'))
 	Write-Debug $parts[1]
 	if ($dev.Antecedent -match 'componentID=""') {
 		$xpathstr = "//SoftwareComponent[@packageType='LWXP']/SupportedDevices/Device/PCIInfo"
 		if ($componentType -match "DRVR") {
 			$xpathstr += "[@" + $parts[2] + " and @" + $parts[3] + "]/../../.."
 			$xpathstr += "/SupportedOperatingSystems/OperatingSystem[@osVendor=`'Microsoft`' and @osCode=`'" + $osCodeID + "`']/../.."
 		} else {
 			$xpathstr += "[@" + $parts[2] + " and @" + $parts[3] + " and @" + $parts[4] + " and @" + $parts[5] + "]/../../.."
 			#$xpathstr += "/SupportedSystems/Brand[@prefix=`'" + $model[0] + "`']/Model[@systemID=`'" + $systemID + "`']/../../.."
 			$xpathstr += "/ComponentType[@value='FRMW']/.."
 			
 		}
 		$xpathstr += "/ComponentType[@value=" + $componentType + "]/.."
 	} else {
 		$xpathstr = "//SoftwareComponent[@packageType='LWXP']/SupportedDevices/Device[@"	
 		$xpathstr += $parts[0].substring($parts[0].indexof("componentID"))
 		$xpathstr += "]/../../SupportedSystems/Brand[@prefix=`'" + $model[0] + "`']/Model[@systemID=`'"
 		$xpathstr += $systemID + "`']/../../.."
 	}
 	Write-Debug $xpathstr
 	
 	$result = Select-Xml $catalog -XPath $xpathstr |Select-Object -ExpandProperty Node
 }
`

