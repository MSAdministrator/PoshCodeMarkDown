---
Author: kevmar
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5179
Published Date: 2014-05-22t03
Archived Date: 2014-05-24t12
---

# get pages printed - 

## Description

it parses the print log file for printer events related to printing. it is important that the log be enabled. if a log is cleared, then this data will be inaccurate.

## Comments



## Usage



## TODO



## script

`get-printhistory`

## Code

`#
 #
 
 <#
 .Synopsis
    Pulls the printer history from the target computer.
 .DESCRIPTION
    It parses the print log file for printer events related to printing. It is important that the log be enabled. If a log is cleared, then this data will be inaccurate.
 .PARAMETER computername
 The computer name(s) to retrieve the info from.
 .EXAMPLE
    Get-PrintHistory
 .EXAMPLE
    Get-PrintHistory -ComputerName localhost
 #>
 function Get-PrintHistory
 {
     [CmdletBinding()]
     [OutputType([object])]
     Param
     (
         [Parameter(Mandatory=$false,
                    ValueFromPipelineByPropertyName=$true,
                    Position=0)]
         [string]$ComputerName="localhost"
 
     )
 
     Begin
     {
     }
     Process
     {
         Write-Verbose "Getting Events from Computer: $ComputerName"
         $log = Get-WinEvent -FilterHashTable @{ "LogName"= "Microsoft-Windows-PrintService/Operational";"ID"="307"} -ComputerName $ComputerName
       
         $MessageRegEx = "(?<Document>.+), Print Document owned by (?<Username>.+) on (?<Computer>.+) was printed on (?<Printer>.+) through port (?<IP>.+)\.  Size in bytes: (?<Size>\d+)\. Pages printed: (?<Pages>\d+)\. No user action is required\."
         Write-Verbose "Parsing $($log.Count) events"
         $log | ?{$_.message -match $MessageRegEx} | 
             %{ New-Object PSObject -property @{"Document"=$Matches.Document;
             "UserName"=$Matches.Username;
             "IP"=$Matches.IP;
             "ComputerName"=$Matches.Computer;
             "Pages"=$Matches.Pages;
             "TimeStamp"= $_.TimeCreated;
             "Printer" = $Matches.Printer;
             "PrintHost" = $_.MachineName
             }}
 
     }
     End
     {
         Write-Verbose "Done processesing print events"
     }
 }
`

