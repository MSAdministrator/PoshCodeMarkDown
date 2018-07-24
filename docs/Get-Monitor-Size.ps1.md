---
Author: adam bertram
Publisher: 
Copyright: 
Email: 
Version: 2.54
Encoding: ascii
License: cc0
PoshCode ID: 4688
Published Date: 2015-12-11t03
Archived Date: 2015-04-15t22
---

# get monitor size - 

## Description

this script queries wmi to find basic monitor size information.  it then performs some math on these attributes to come up with the size of all monitors attached to a local or remote device.

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 param($ComputerName = 'COMPUTERNAME')
 
 $output = [PSCustomObject]@{ComputerName = $ComputerName;MonitorSizes=''}
 
 $oWmi = Get-WmiObject -Namespace 'root\wmi' -ComputerName $ComputerName -Query "SELECT MaxHorizontalImageSize,MaxVerticalImageSize FROM WmiMonitorBasicDisplayParams";
 $sizes = @();
 if ($oWmi.Count -gt 1) {
 	foreach ($i in $oWmi) {
 		$x = [System.Math]::Pow($i.MaxHorizontalImageSize/2.54,2)
 		$y = [System.Math]::Pow($i.MaxVerticalImageSize/2.54,2)
         $sizes += [System.Math]::Round([System.Math]::Sqrt($x + $y),0)
 } else {
 	$x = [System.Math]::Pow($oWmi.MaxHorizontalImageSize/2.54,2)
 	$y = [System.Math]::Pow($oWmi.MaxVerticalImageSize/2.54,2)
 	$sizes += [System.Math]::Round([System.Math]::Sqrt($x + $y),0)
 
 $output.MonitorSizes = $sizes
 
 $output
`

