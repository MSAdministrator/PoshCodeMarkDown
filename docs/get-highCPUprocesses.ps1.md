---
Author: internetrush1
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6156
Published Date: 2016-12-29t14
Archived Date: 2016-03-18t21
---

# get-highcpuprocesses - 

## Description

enumerates all cpu related tasks (filters out null cpu times) with the following properties

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
     param([string]$computerName, [System.Management.Automation.PSCredential]$Credential)
 
     invoke-command -ComputerName $computername -Credential $Credential -ScriptBlock {
 
         $CPUPercent = @{
                 Name = 'CurrentCPUPercent'
                 Expression = {
                 $TotalSec = (New-TimeSpan -Start $_.StartTime).TotalSeconds
                 [Math]::Round( ($_.CPU * 100 / $TotalSec), 2)
             }
         }
         $processes = Get-Process | ? {$_.TotalProcessorTime -ne $Null} | Select-Object -Property Name, @{Name="CPUSeconds"; Expression = {($_.CPU)}},@{Name="WorkingSetMB"; Expression = {($_.WorkingSet / 1mb)}},TotalProcessorTime, $CPUPercent, Description 
     
         $processes | Sort-Object -Property CurrentCPUPercent -Descending | Out-GridView -Title 'Current Highest CPU %'
     }
 }
`

