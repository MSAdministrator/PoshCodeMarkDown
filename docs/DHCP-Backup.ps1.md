---
Author: r derickson
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6309
Published Date: 2016-04-19t13
Archived Date: 2016-04-22t03
---

# dhcp backup - 

## Description

discovers dhcp online servers and if systemroot\windows\system32\dhcp\backup exists, copies the folder on each server to a network share (\\network\share\hostname).

## Comments



## Usage



## TODO



## module

`get-onlinedhcpservers`

## Code

`#
 #
 Import-Module ActiveDirectory
 
 param (
     [Parameter(Position=1)]
     $searchBase = "cn=configuration,dc=domain,dc=com",
     [Parameter(Position=2)]
     $backupDestRoot = "\\network\share\"
 )
 
 function Get-OnlineDhcpServers {
     param (
         [Parameter(Mandatory=$true,Position=1)]
         $dhcpSearchBase
         )
     $arrDhcpServers = Get-ADObject -SearchBase $dhcpSearchBase -Filter "objectclass -eq 'dhcpclass' -AND name -ne 'dhcproot'"
     ForEach ($dhcpServer in $arrDhcpServers) {
         if (!(Test-Connection -ComputerName $dhcpServer.name -Count 2 -Quiet -ErrorAction SilentlyContinue)) {
             $arrDhcpServers = @($arrDhcpServers | Where-Object {$_.name -ne $dhcpServer.name})
         }
     }
     return $arrDhcpServers
 }
 
 function Backup-DhcpServers {
     param (
         [Parameter(Mandatory=$true,Position=1)]
         $arrDhcpBackupSrcNames,
         [Parameter(Mandatory=$true,Position=2)]
         $dhcpBackupDestRoot
     )
     ForEach ($dhcpBackupSrcName in $arrDhcpBackupSrcNames) {
         $dhcpBackupSrc = "\\" + $dhcpBackupSrcName + "\c$\Windows\System32\Dhcp\Backup"
         $dhcpBackupDest = $dhcpBackupDestRoot + $dhcpBackupSrcName
         if (Test-Path -Path $dhcpBackupSrc) {
             Remove-Item -Path $dhcpBackupDest -Recurse -Force -Verbose
             New-Item -Path $dhcpBackupDest -ItemType Directory
             Copy-Item -Path $dhcpBackupSrc -Destination $dhcpBackupDest -Recurse -Verbose
         }
     }
 }
 
 $onlineDhcpServers = Get-OnlineDhcpServers -dhcpSearchBase $searchBase
 $onlineDhcpServerNames = $onlineDhcpServers | ForEach-Object {$_.name.Split(".")[0]}
 Backup-DhcpServers -arrDhcpBackupSrcNames $onlineDhcpServerNames -dhcpBackupDestRoot $backupDestRoot
`

