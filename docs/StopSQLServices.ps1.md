---
Author: rob sewell http
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 4165
Published Date: 
Archived Date: 2017-03-16t01
---

# stopsqlservices.ps1 - stopsqlservices.ps1

## Description

#############################################################################################

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 #############################################################################################
 #
 #
 
 $Server= Read-Host "Please Enter the Server - This WILL stop all SQL services"
 
 get-service -ComputerName $server|Where-Object { $_.Name -like '*SQL*' }
 $Services = Get-Service -ComputerName $server|Where-Object { $_.Name -like '*SQL*' -and $_.Status -eq 'Running' -and $_.Name -ne 'SQLSERverAGENT'}
 
 foreach($Service in $Services)
 {
 if($service.Status -eq 'Running')
 {
 $ServiceName = $Service.displayname
 (get-service -ComputerName $Server  -Name $ServiceName).Stop()
  while((Get-Service -ComputerName $server -Name $ServiceName).status -ne'Stopped')
  }
  }
 get-service -ComputerName $server|Where-Object { $_.Name -like '*SQL*' }
`

