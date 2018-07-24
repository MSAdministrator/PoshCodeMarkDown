---
Author: leon scheltema
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 4187
Published Date: 2013-05-29t13
Archived Date: 2013-06-13t05
---

# whereismyvm - 

## Description

script to find on which host a particular vm is running when vcenter is down

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 $ScriptPath = (Split-Path ((Get-Variable MyInvocation).Value).MyCommand.Path)
 $ContentFolder = $ScriptPath + "\Lists\"
 
 "**************************************"
 
 "*                                    *"
 
 "*       Where is My VM ?             *"
 
 "*                                    *"
 
 "*   Please Select the Cluster        *"
 
 "*      You want to search            *"
 
 "*                                    *"
 
 "*       1) Cluster_1                *"
 
 "*       2) Cluster_2                 *"
 
 "*       3) Cluster_3                 *"
 
 "*       4) Complete_Datacenter       *"
 
 "*       5) All_active_hosts          *"
 
 "*       6) Exit                      *"
 
 "*                                    *"
 
 "*                                    *"
 
 "**************************************"
 
 $a = read-host "Select the Cluster"
 
 IF ($a -eq 1){$List = $ContentFolder + "\Cluster_1.txt"}
 
 Elseif ($a -eq 2){$List = $ContentFolder + "\Cluster_2.txt"}
 
 Elseif ($a -eq 3){$List = $ContentFolder + "\Cluster_3.txt"}
 
 Elseif ($a -eq 4){$List = $ContentFolder + "\Complete_Datacenter.txt"}
 
 Elseif ($a -eq 5){$List = $ContentFolder + "\All_active_hosts.txt"}
 
 Elseif ($a -eq 6){exit}
 
 $findvm = Read-Host -Prompt "Missing VM Name"  
 $esxservers = get-content $List 
 foreach ($esxserver in $esxservers) {Connect-VIServer  -WarningAction SilentlyContinue -Server $esxserver -User username -Password password
 $vmlist = (Get-VM | select -expandproperty name)
 foreach ($vm in $vmlist) 
     {
 	if ($vm -eq $findvm){Write-Host $findvm "was located on $esxserver"}	
 
     if ($vm -eq $findvm){exit}
 
     Elseif ($vm -eq $findvm){Write-Host "$findvm was not located on host"}
     } 
 }
 Write-Host "$findvm was not located on any host"
 Disconnect-VIServer -server $esxserver -Force -Confirm:$false
`

