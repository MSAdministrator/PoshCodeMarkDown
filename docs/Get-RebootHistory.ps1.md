---
Author: fishee
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: utf-8
License: cc0
PoshCode ID: 4440
Published Date: 2013-09-04t16
Archived Date: 2016-10-30t00
---

# get-reboothistory - 

## Description

http

## Comments



## Usage



## TODO



## function

`get-reboothistory`

## Code

`#
 #
 function Get-RebootHistory{
     <#
     .SYNOPSIS
     PSTip Get your reboot history.
     .DESCRIPTION
     Have you ever wondered how often is your station rebooted? Let�s ask the Windows Event Log and get time of last five reboots. You will use the Get-WinEvent cmdlet to connect to System event log. You are interested in �The Event log service was started.� event which has Id 6005.
     .EXAMPLE
     Get-RebootHistory
 
     PS C:> .\Get-RebootHistory.ps1
 
     ComputerName          Reboots
     ------------          -------
     TEST11122             {System.Diagnostics.Eventing.Reader.EventLogRecord, Syst...
 
 
     .NOTES
     There is of course a 6006 event if you are more interested in shutdowns.
     Since operating sytstems can halt or a power failure may occur, the 1074 may not be present in the system log, making false impression nothing has happened.
     .LINK
     http://www.powershellmagazine.com/2012/10/15/pstip-get-your-reboot-history/?utm_source=feedburner&utm_medium=email&utm_campaign=Feed%3A+PowershellMagazine+%28PowerShell+Magazine%29
     #>
     [cmdletbinding()]
     param(
         [Parameter(Mandatory=$False,ValueFromPipeline=$true)]
         [string[]]$ComputerName=$env:COMPUTERNAME,
 
         [ValidateSet(6005,6006)]
         [int]$EventID=6005,
 
         [ValidateRange(1,[uint32]::MaxValue)]
         [int]$MaxEvents=0
     )
 
     begin{
         $xml=@"
 <QueryList>
 <Query Id="0" Path="System">
 <Select Path="System">*[System[(EventID=$EventID)]]</Select>
 </Query>
 </QueryList>
 "@
     $EventArgs = @{FilterXml=$xml;ErrorAction='Stop'}
     if($MaxEvents){
         $EventArgs['MaxEvents'] = $MaxEvents
     }
 }
     process{
         foreach($Computer in $ComputerName){
             try{
                 Write-Output ([PSCustomObject]@{ComputerName=$Computer;Reboots=(Get-WinEvent -ComputerName $Computer @EventArgs)})
             }
             catch{
                 Write-Error "Could not get reboot history from '$Computer'. $($Error[0])"
             }            
         }
     }
     end{}
 }
`

