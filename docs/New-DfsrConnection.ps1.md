---
Author: j palmero
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 4424
Published Date: 2015-08-28t16
Archived Date: 2015-08-02t04
---

# new-dfsrconnection - 

## Description

script creates one-way connections between one sending member server and a list of receiving member servers. the group schedule is used.

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 $array = "SITE01","SITE02","SITE03","SITE04","SITE05"
 
 $repGroup  = Get-DfsrReplicationGroup -Name "ReplicationGroupName"
 
 $sendMember = $repGroup | Get-DfsrMember -ComputerName SITE00
 
 ForEach ($site in $array) {
     $site = $site + "COMPANY"
     Write-Host "Adding connection for:" $site
     
     $receiveMember = $repGroup | Get-DfsrMember -ComputerName $site
       
     New-DfsrConnection -Member $receiveMember -SendingMember $sendMember -Enabled $true 
 }
`

