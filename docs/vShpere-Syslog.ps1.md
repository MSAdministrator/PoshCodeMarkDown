---
Author: cory murdoch
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5320
Published Date: 2014-07-22t00
Archived Date: 2014-07-25t21
---

# vshpere syslog - 

## Description

script for vsphere syslog collector. uses ip address from directories created for each esxi host to do a dns lookup and create shortcuts for those folders based on the returned dns of the esxi host

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 ########################################################################################################
 ########################################################################################################
 
 
 Function ReverseDNS
 {
 	$nslookup = "nslookup " + $args[0] + " " + $ns + " 2> null"
 	$result = Invoke-Expression ($nslookup) 
 	$global:reverse_solved_ip = $result.SyncRoot[3]
 
 	if ($result.count -lt 4) 
 	{
 		$global:reverse_solved_ip = "No record found"
 	}
 	else
 	{
 		$global:reverse_solved_ip = $global:reverse_solved_ip.Remove(0,9)
 	}
 }
 
 $ns = "dns.server.com"
 
 $folderpath = "C:\ProgramData\VMware\VMware Syslog Collector\Data\"
 
 $foldername = Get-ChildItem $folderpath | Where-Object {$_.PSIsContainer} | Foreach-Object {$_.Name}
 
 Foreach ($ip in $foldername){
 	
 	
 	ReverseDNS $ip
 	
 	Write-Host $ip
 	Write-Host $reverse_solved_ip
 	Write-Host ""
 	
 	
 	
 	if ($reverse_solved_ip -ne "No record found") {
 		
 		$target = $folderpath + $ip
 		$shortcut = $folderpath + $reverse_solved_ip + ".lnk"
 		$wshshell = New-Object -ComObject WScript.Shell
 		$link = $wshshell.CreateShortcut($shortcut)
 		$link.TargetPath = $target
 		$link.Save()
 	}
 }
`

