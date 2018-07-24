---
Author: adam bertram
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 4683
Published Date: 2013-12-09t18
Archived Date: 2013-12-13t12
---

# get-devicechassis - 

## Description

a simple function that returns where a remote windows device is a thin client or a desktop.

## Comments



## Usage



## TODO



## function

`get-devicechassis`

## Code

`#
 #
 Function Get-DeviceChassis () {
     [CmdletBinding()]
     Param($ComputerName = 'localhost')
  
     PROCESS { 
         $Output = New-Object PsObject -property @{ComputerName = $ComputerName;Chassis=''}
         try {
             $Model = (Get-WmiObject -Query "SELECT Model FROM Win32_ComputerSystem" -ComputerName $ComputerName).Model;
                 if (($Model -like 'hp t*') -and ($Model -notlike 'hp touchsmart*')) {
                     Write-Verbose "Found chassis to be thin client via WMI";
                     $Output.Chassis = 'Thin Client'
 	    	} else {
                     Write-Verbose "Found chassis to be desktop via WMI";
                     $Output.Chassis = 'Desktop'
         } catch {
             if ((Get-Item "\\$ComputerName\admin$\explorer.exe").Attributes -match 'Compressed') {
                 Write-Verbose "Found chassis to be thin client via compressed files";
                 $Output.Chassis = 'Thin Client'
             } else {
                 Write-Verbose "Found chassis to be desktop via compressed files";
                 $Output.Chassis = 'Desktop'
             }
         } finally {
             $Output
         }
     }
 }
`

