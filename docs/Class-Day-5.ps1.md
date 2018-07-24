---
Author: don jones
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5071
Published Date: 2014-04-11t13
Archived Date: 2014-07-01t20
---

# class day 5 - 

## Description

day 5 example includes supportsshouldprocess

## Comments



## Usage



## TODO



## function

`get-shareinfo`

## Code

`#
 #
  
  function Get-ShareInfo { 
  <#
  .SYNOPSIS
  Gets share info. Name really says it all.
  .DESCRIPTION
  Not much more to offer. This uses CIM, so it needs Remoting enabled.
  .PARAMETER ComputerName
  The name of the computer(s) to query. Needs to be real names, not IP addresses.
  .PARAMETER ErrorLogFile
  Path and name of a text file where failed computer names will be logged. Might
  be deleted each time you run the function.
  .EXAMPLE
  Get-ShareInfo -ComputerName ONE,TWO,THREE
  .EXAMPLE
  "one","two" | Get-ShareInfo
  This example demostrates the use of pipeline input.
  #>
     [CmdletBinding()]
     param(
         [Parameter(Mandatory=$True,HelpMessage='The computer name, duh',ValueFromPipeline=$True)]
         [string[]]$ComputerName,
 
         [Parameter(HelpMessage='A log file name')]
         [string]$ErrorLogFile = 'c:\errors.txt'
     )
     BEGIN {
         Remove-Item -Path $ErrorLogFile -force -ErrorAction SilentlyContinue
     }
     PROCESS {
         foreach ($computer in $ComputerName) {
             Write-Verbose "Connecting to $computer"
 
             try {
                 $shares = Get-CimInstance -ErrorVariable x -ErrorAction Stop -ClassName Win32_Share -ComputerName $Computer -filter "Path<>''"
                 Write-Verbose "Got $($shares.count) shares"
 
                 Write-Verbose "Enumerate the shares on $computer"
                 foreach ($share in $shares) {
 
                     $sharename = $share.path -replace "\\","\\"
                     $directory = Get-CimInstance -EA Stop -ClassName Win32_Directory -Filter "Name='$($sharename)'"
 
                     Write-Debug "Finished getting info for $computer; building output object"
                     $properties = @{'ComputerName'=$Computer;
                                     'Name'=$share.name;
                                     'Path'=$share.path;
                                     'Caption'=$share.caption;
                                     'Readable'=$directory.readable;
                                     'Writable'=$directory.writable}
                     $output = New-Object -TypeName PSObject -Property $properties
                     $output.psobject.typenames.insert(0,'ConTech.ShareInfo')
 
                     Write-Debug "Finished building output for $computer, ready to write"
                     Write-Output $output
 
             } catch {
                 $computer | out-file $ErrorLogFile -Append
                 Write-Warning "$computer - $x"
 
 
 
 function Set-ServicePassword {
     [CmdletBinding(SupportsShouldProcess=$True,ConfirmImpact='Medium')]
     Param(
         [Parameter(Mandatory=$True,ValueFromPipeline=$True,ValueFromPipelineByPropertyName=$True)]
         [string[]]$ComputerName,
 
         [Parameter(Mandatory=$True,ValueFromPipelineByPropertyName=$True)]
         [string]$ServiceName,
 
         [Parameter(Mandatory=$True,ValueFromPipelineByPropertyName=$True)]
         [string]$NewPassword
     )
     PROCESS {
         foreach ($computer in $computername) {
             Write-Verbose "Changing $ServiceName password on $Computer"
 
             if ($PSCmdlet.ShouldProcess("service '$servicename' on computer: $computer")) {
                 $ret = Get-CimInstance -ClassName Win32_Service `
                                        -Filter "Name='$ServiceName'" `
                                        -ComputerName $computer |
                        Invoke-CimMethod -MethodName Change `
                                         -Arguments @{'StartPassword'=$NewPassword}
 
                 if ($ret.returnvalue -ne 0) {
                     Write-Warning "Error value $ret for $ServiceName on $Computer"
                 }
             }
 
         }
     }
 }
`

