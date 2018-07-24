---
Author: 4wheels
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 4054
Published Date: 2015-03-27t22
Archived Date: 2015-05-07t23
---

# map network drive - 

## Description

map network drive using powershell to call wscript.network. psdrive just does not act the same as a mapped network drive. i finally wrote these to connect to specific servers and directories to move files around.

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 #
 
 	$Drive = "O:"
 	$UNC = "\\ampwcsqlsvr2\nam401k"
 cls
 
 	
 	if (((New-Object -Com WScript.Network).EnumNetworkDrives() | Where-Object `
 {$_ -eq $Drive})) 
 	
 	$net = $(New-Object -comobject WScript.Network)
 	$net.RemoveNetworkDrive($Drive,1)
 	
 	
 	} 
 if (!((New-Object -Com WScript.Network).EnumNetworkDrives() | Where-Object `
 {$_ -eq $Drive}))
 		{
 		$net = $(New-Object -comobject WScript.Network)
 		$net.mapnetworkdrive($Drive,$Unc) 
 		}
 
 explorer.exe $Drive
`

