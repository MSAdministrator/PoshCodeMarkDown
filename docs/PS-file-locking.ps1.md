---
Author: michael r
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: utf-8
License: cc0
PoshCode ID: 1566
Published Date: 2012-01-11t12
Archived Date: 2012-12-29t04
---

# ps file locking - 

## Description

the code uses a file locking mechanism to ensure that the

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 #################################################
 #
 
 ############################################
 
 $lockfile = �d:\lock.lck�
 $lockstatus = 0
 While ($lockstatus -ne 1)
 {
 	If (Test-Path $lockfile)
 	{
 		echo �Lock file found!�
 		$pidlist = Get-content $lockfile
 		If (!$pidlist)
 		{
 			$PID | Out-File $lockfile
 			$lockstatus = 1
 		}
 		$currentproclist = Get-Process | ? { $_.id -match $pidlist }
 		If ($currentproclist)
 		{
 			echo �lockfile in use by other process!�
 			$rndwait = New-Object system.Random
 			$rndwait=	$rndwait.next(500,1000)
 			echo �Sleeping for $rndwait Milliseconds�
 			Start-Sleep -Milliseconds $rndwait
 		}
 		Else
 		{
 			Remove-Item $lockfile -Force
 			$PID | Out-File $lockfile
 			$lockstatus = 1
 		}
 	}
 	Else
 	{
 		$PID | Out-File $lockfile
 		$lockstatus = 1
 	}
 }
 
 #��������������������������������
 echo �Hi, it seems that no other script is doing the same as me now.. so I can do my job exclusively�
 
 #��������������������������������
 Remove-Item $lockfile -Force
`

