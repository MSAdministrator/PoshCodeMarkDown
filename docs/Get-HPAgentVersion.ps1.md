---
Author: chad miller
Publisher: 
Copyright: 
Email: 
Version: 1.0
Encoding: ascii
License: cc0
PoshCode ID: 4010
Published Date: 2013-03-13t13
Archived Date: 2013-03-21t05
---

# get-hpagentversion - 

## Description

the get-hpagentversion function gets the hp psp/spp version.

## Comments



## Usage



## TODO



## script

`get-hpagentversion`

## Code

`#
 #
 #######################
 <#
 .SYNOPSIS
 Gets HP PSP/SPP Agent Version
 .DESCRIPTION
 The Get-HPAgentVersion function gets the HP PSP/SPP version.
 .EXAMPLE
 Get-HPAgentVersion "Z002"
 This command gets information for computername Z002.
 .EXAMPLE
 Get-Content ./servers.txt | Get-HPAgentVersion
 This command gets information for a list of servers stored in servers.txt.
 .EXAMPLE
 Get-HPAgentVersion (get-content ./servers.txt)
 This command gets information for a list of servers stored in servers.txt.
 .NOTES 
 Version History 
 v1.0   - Chad Miller - Initial release 
 #>
 function Get-HPAgentVersion
 {
     [CmdletBinding()]
     param(
     [Parameter(Mandatory=$true,ValueFromPipeline=$true,ValueFromPipelineByPropertyName=$true)]
     [ValidateNotNullorEmpty()]
     [string[]]$ComputerName
     )
     BEGIN {}
     PROCESS {
         foreach ($computer in $computername) {
             Get-WMIObject -ComputerName $computer -Query "SELECT * FROM CIM_DataFile WHERE Drive ='C:' AND Path='\\WINDOWS\\system32\\CpqMgmt\\' AND FileName='agentver' AND Extension='dll'"  |
             Select CSName, Version, @{n='OS';e={(gwmi win32_operatingsystem -ComputerName $_.CSName).Version}}
         }
     }
     END {}
 }
`

