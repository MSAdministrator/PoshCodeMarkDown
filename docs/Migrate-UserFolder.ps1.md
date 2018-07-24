---
Author: chriskenis
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 3549
Published Date: 2013-07-27t00
Archived Date: 2016-05-31t08
---

# migrate userfolder - 

## Description

user home drive migration script (in progress but nearly complete)

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 param(
 $RootFolder = "\\server1\users",
 $NewRootFolder = "\\server2\users",
 $LogFolder = "C:\Projects\HomeDirs",
 $NewSubFolders = @("Documents","Favorites","Desktop","Links","Contacts"),
 $domain = "domain",
 [switch]$SetACL
 )
 $UserFolders = gci -Path $RootFolder | ?{$_.PSIsContainer}
 $UserFolders | foreach-object -Process {
 	$NewUserFolder =  $NewRootFolder + "\" + $_.name
 	New-Item -Path $NewUserFolder -ItemType "Directory"
 	if ($SetACL){
 		$acl = Get-Acl $NewUserFolder
 		$Owner = New-Object System.Security.Principal.NTAccount($domain, $_.name)
 		$rule = New-Object System.Security.AccessControl.FileSystemAccessRule($Owner,"Modify", "ContainerInherit, ObjectInherit", "None", "Allow")
 		$acl.SetAccessRule($rule)
 		Set-Acl $NewUserFolder $acl
 		}
 	$NewSubFolders | foreach{New-Item -Path $($NewUserFolder + "\" + $_) -type directory}
 	$LogFile = $LogFolder + "\" + $_.name + ".log"
 	$JobName = $_.name + "_RCsync"
 	$CommandString = "robocopy $($_.FullName) $($NewUserFolder + "\Documents") /COPYALL /MIR /FFT /Z /Log+:$LogFile"
 	start-job -Scriptblock {invoke-Expression $input} -name $JobName -InputObject $CommandString
     }
 Get-Job | Wait-Job
 Get-Job | Receive-Job
`

