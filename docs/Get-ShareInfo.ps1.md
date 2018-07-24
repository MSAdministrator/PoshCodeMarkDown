---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5068
Published Date: 
Archived Date: 2014-04-13t11
---

#  - 

## Description

class lab 8

## Comments



## Usage



## TODO



## function

`get-shareinfo`

## Code

`#
 #
  function Get-ShareInfo {
     [CmdletBinding()]
     param(
         [Parameter(Mandatory=$True,HelpMessage='The computer name, duh',ValueFromPipeline=$True)]
         [string[]]$ComputerName,
 
         [Parameter(HelpMessage='A log file name')]
         [string]$ErrorLogFile = 'c:\errors.txt'
     )
     PROCESS {
         foreach ($computer in $ComputerName) {
             Write-Verbose "Connecting to $computer"
             $shares = Get-CimInstance -ClassName Win32_Share -ComputerName $Computer -filter "Path<>''"
 
             foreach ($share in $shares) {
 
                 $sharename = $share.path -replace "\\","\\"
                 $directory = Get-CimInstance -ClassName Win32_Directory -Filter "Name='$($sharename)'"
 
                 $properties = @{'ComputerName'=$Computer;
                                 'Name'=$share.name;
                                 'Path'=$share.path;
                                 'Caption'=$share.caption;
                                 'Readable'=$directory.readable;
                                 'Writable'=$directory.writable}
                 $output = New-Object -TypeName PSObject -Property $properties
                 Write-Output $output
 
 
 
 
 "dc","win81" | Get-ShareInfo -Verbose | export-csv output.csv
`

