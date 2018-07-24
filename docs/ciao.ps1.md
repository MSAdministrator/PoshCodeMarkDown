---
Author: brian english
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 3041
Published Date: 2011-11-08t06
Archived Date: 2011-11-16t01
---

# ciao - 

## Description

a vi toolkit script to update all vm guests on the selected hosts

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 ########################################################
 ########################################################
 ########################################################
 ########################################################
 
 if ($args[0] -eq $null)
 {$hostName = Read-Host "What host do you want to connect to?"}
 else
 {$hostName = $args[0]}
 
 Connect-VIServer $hostName
 
 ########################################################
 $vms = get-vm
 
 ########################################################
 
 Foreach ($i in $vms) 
   if ((get-vm -name $i).Name -eq $hostname)
   {"Skipping " + $hostname}
   #{"Skipping DNS/DC/DHCP server name"}
   else
   { 
     if ((get-vm -name $i).PowerState -eq "PoweredOn")
     { $i
       Get-Date -format "hh:mm:ss"
       update-tools -guest (get-vmguest -vm (get-vm -name $i))
       Get-Date -format "hh:mm:ss"
`

