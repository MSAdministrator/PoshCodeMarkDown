---
Author: bassamf
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2228
Published Date: 2011-09-11t20
Archived Date: 2017-02-18t07
---

# inovke command exclusive - 

## Description

this invokes a command exclusively

## Comments



## Usage



## TODO



## function

`invoke-commandex`

## Code

`#
 #
 function Invoke-CommandEx
 {
 	param
 	(
 		[String] $MutexName = $(throw "Name is required."),
 		[ScriptBlock] $Scriptblock = $(throw "Scriptblock is required."),
 		[Object[]] $ArgumentList
 	)
 	$MutexWasCreated = $false;
 	$Mutex = $null;
 	Write-Host "Waiting to acquire lock [$MutexName]..." -f Cyan
 	[System.Reflection.Assembly]::LoadWithPartialName("System.Threading");
 	try {
 		$Mutex = [System.Threading.Mutex]::OpenExisting($MutexName);
 	}catch{
 		$Mutex = New-Object System.Threading.Mutex($true,$MutexName,[ref]$MutexWasCreated);
 	}
 	try {
 		if(!$MutexWasCreated){
 			$Mutex.WaitOne() | Out-Null;
 		}
 	}catch{}
 	Write-Host "Lock [$MutexName] acquired. Executing..." -f Cyan
 	foreach($arg in $ArgumentList){
 		$params += "`"$arg`" ";
 	}
 	$cmd = @"
 	function f{
 		$Scriptblock
 	}
 	f $params
 "@;
 	Invoke-Expression $cmd
 	
 	Write-Host "Releasing lock [$MutexName]..." -f Cyan
 	try {
 		$Mutex.ReleaseMutex() | Out-Null;
 	}catch{}
 }
`

