---
Author: hotsnoj
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 3061
Published Date: 2011-11-21t08
Archived Date: 2011-11-29t18
---

# get-lockedtime - 

## Description

finds the length of time a session has been “locked”.

## Comments



## Usage



## TODO



## function

``

## Code

`#
 #
 param (
     [parameter(position=0,
         ValueFromPipeline = $true,
         ValueFromPipelineByPropertyName = $true)]
     $ComputerName
 )
 
 Begin {
 	$alreadybegan = $false;
 	
     function WmiDateToDotnet {
         param($date);
         
         if($date.length -ne 25) {
             throw New-Object ArgumentOutOfRangeException;
         }
         
         return New-Object datetime @(([int]$date.substring(0,4)), ([int]$date.substring(4,2)), ([int]$date.substring(6,2)), ([int]$date.substring(8,2)), ([int]$date.substring(10,2)), ([int]$date.substring(12,2)), ([datetimekind]::Local))
     }
     
     
     function getData {
         param($cn);
             
         $qwinsta_res = qwinsta.ps1 $cn
         $processes = gwmi -Class win32_process -ComputerName $cn
 
         $logonuis = $processes | ?{$_.Name -match "logonui"} 
         for($i = 0; $i -lt $logonuis.Length; $i++) {
             Add-Member -Force -in $logonuis[$i] -MemberType NoteProperty -Name OwnerData -Value ($qwinsta_res | ?{$_.ID -eq $logonuis[$i].SessionId})
             Add-Member -Force -in $logonuis[$i] -MemberType ScriptProperty -Name Owner -Value {return $this.OwnerData.UserName;}
             Add-Member -Force -in $logonuis[$i] -MemberType NoteProperty -Name LockTime -Value (WmiDateToDotnet $logonuis[$i].creationdate)
         }
         
         $logonuis
     }
     
     if($ComputerName -ne $null) {
 		$alreadybegan = $true;
         getData $ComputerName;
     }
 }
 
 Process {
 	if($alreadybegan -eq $true) { continue; }
 	
     if($_ -eq $null) {
         throw New-Object exception "not a valid object to get a computer name from";
     } elseif($_.gettype().equals([string])) {
         $cn = $_;
     } elseif($_.computername -ne $null) {
         $cn = $_.computername
     } elseif($_.machinename -ne $null) {
         $cn = $_.machinename
     } elseif($_.hostname -ne $null) {
         $cn = $_.hostname
     } else {
         throw New-Object exception "not a valid object to get a computer name from";
     }
     
     getData $cn;
 }
`

