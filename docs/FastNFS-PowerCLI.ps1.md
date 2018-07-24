---
Author: eprich
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2735
Published Date: 2011-06-14t12
Archived Date: 2011-06-17t15
---

# fastnfs - powercli - 

## Description

mounts nfs datastore to a group of esx hosts.

## Comments



## Usage

enter a list of esx hosts (by ip or hostname). then enter the ip, path, and datastore name of the share.

## TODO



## script

``

## Code

`#
 #
 #
 #
 $nfssrv = Read-Host "Enter the name or IP of the NFS server"
  
 $nfspath = Read-Host "Enter the full path to the share"
  
 $nfsds = Read-Host "Enter a name for the new NFS datastore"
  
 $esx = @()
  
 do {
     $input = Read-Host "Add an ESX host by FQDN or IP"
     if ($input -ne ""){
         $esx += $input
     }
    }
 until ($input -eq "")
  
 foreach ($name in $esx)
         {
         New-Datastore -VMHost $name -Name $nfsds -Path $nfspath -Nfshost $nfssrv -Nfs
         }
`

