---
Author: don jones
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5067
Published Date: 2014-04-09t18
Archived Date: 2014-04-13t11
---

# step 1 - 

## Description

lab 7

## Comments



## Usage



## TODO



## function

`get-shareinfo`

## Code

`#
 #
  function Get-ShareInfo {
     param(
         [string[]]$ComputerName,
 
         [string]$ErrorLogFile
     )
 
     foreach ($computer in $ComputerName) {
 
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
 
 
 
 
 Get-ShareInfo -ComputerName dc,win81 -ErrorLogFile c:\errors.txt
`

