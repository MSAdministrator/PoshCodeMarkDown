---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 3860
Published Date: 
Archived Date: 2013-01-09t07
---

#  - 

## Description

this is a very quick and dirty script to import non-exported machines into hyper-v. for example if you have a data disk with vm’s and your system disk fails. this can “reimport” the machines without need to recreate the virtual machine config with all the work that implies. switches needs to be recreated in addition to this so network connections will be broken by default. if you have snapshots you will need to add in links and acl�s for that also. not supported and not recommended for production.

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 PARAM(
 	$location=$(throw "Make sure to specify a location for old machines to be imported")
 )
 $oldVM = Get-ChildItem "$location\*.xml"
 
 foreach($vm in $oldVM)
 {
 	$vmGuid = [System.IO.Path]::GetFileNameWithoutExtension($vm.Name)
 	Invoke-Command -ScriptBlock {cmd /c "Mklink `"C:\ProgramData\Microsoft\Windows\Hyper-V\Virtual Machines\$vmGuid.xml`" `"D:\VMConfig\Virtual Machines\$vmGuid.xml`""}
 	Invoke-Command -ScriptBlock {cmd /c "Icacls `"C:\ProgramData\Microsoft\Windows\Hyper-V\Virtual Machines\$vmGuid.xml`" /grant `"NT VIRTUAL MACHINE\$vmGuid`":(F) /L"}
 	Invoke-Command -ScriptBlock {cmd /c "Icacls $location /T /grant `"NT VIRTUAL MACHINE\$vmGuid`":(F)"}
 }
`

