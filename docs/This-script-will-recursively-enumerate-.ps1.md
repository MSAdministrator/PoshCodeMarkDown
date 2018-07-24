---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 1122
Published Date: 
Archived Date: 2009-05-27t16
---

#  - 

## Description

this script will recursively enumerate your entire �server� objects, if they all reside under an ou and get the service tag via wmi for each one of them excluding the vmware guest servers.  this is good if you have a lot of servers and don’t want to spend the time having to go to each one of them to manually get the information.

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 
 $Null = Connect-VIServer Your-VM-Server-Here;
 
 $Servers = New-Object System.Collections.ArrayList
 
 $Null = Get-QADComputer -SearchRoot 'your.domain.com/Path/To/Server/OU' |  Select-Object -property Name | Format-Table -HideTableHeaders| Out-String -Stream | ForEach-Object{$Servers.Add($_.ToString().ToUpper())}
 $Null = Get-VM | Select-Object -property Name | Format-Table -HideTableHeaders | Out-String -Stream | ForEach-Object{$Servers.Remove($_.ToString().ToUpper())}
 
 
 foreach ($Server in $Servers | Sort-Object )
 {
 	ServiceTag = (Get-WmiObject Win32_BIOS -comp ($Server.ToString().Split(' '))[0]).SerialNumber;
 	$Result = New-Object -TypeName psobject;
 	$Result | Add-Member -MemberType NoteProperty -Name "Server Name" ($Server.ToString().Split(' '))[0];
 	$Result | Add-Member -MemberType NoteProperty -Name "Service Tag" $ServiceTag;
 	 
 	Write-Output $Result;
 	
 	
 }
`

