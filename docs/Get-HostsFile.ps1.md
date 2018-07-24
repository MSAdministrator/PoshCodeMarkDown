---
Author: boe prox
Publisher: 
Copyright: 
Email: 
Version: 1.1
Encoding: ascii
License: cc0
PoshCode ID: 2568
Published Date: 2011-03-17t14
Archived Date: 2016-03-18t11
---

# get-hostsfile - 

## Description

this function will allow you to retreive the contents of a hosts file on a local or remote system. this can be used with one or more systems.

## Comments



## Usage



## TODO



## function

`get-hostsfile`

## Code

`#
 #
 Function Get-HostsFile {
 <#
 .SYNOPSIS
     Retrieves the contents of a hosts file on a specified system.
 .DESCRIPTION
     Retrieves the contents of a hosts file on a specified system.
 .PARAMETER ComputerName
     The computers to access.
 .NOTES
     Name: Get-HostsFile
     Author: Boe Prox
     DateCreated: 15Mar2011
 
     1.1 - 2011-03-17 - Jason Archer
         Improved pipeline support (and fixed positional usage).
         Added custom object creation and incremental output (better performance and cleaner code).
         For local host, use local path.
         Added error messages for error conditions.
     1.0 - 2011-03-15 - Boe Prox
         Initial release.
 .LINK
     http://boeprox.wordpress.com
 .EXAMPLE
     Get-HostsFile "server1"
 
     Description
     -----------    
     Retrieves the contents of the hosts file on 'server1'.
 #>
 
 [CmdletBinding()]
 Param(
     [Parameter(Position = 0, ValueFromPipeline = $True, ValueFromPipelineByPropertyName = $True)]
     [ValidateNotNull()]
     [string[]]$ComputerName = "localhost"
 )
 
 Begin {
     $PSBoundParameters.GetEnumerator() | Foreach-Object {  
         Write-Verbose "Parameter: $_" 
     }
 }
 
 Process {
     Write-Verbose "Starting process of computers"
     ForEach ($c in $ComputerName ) {
         Write-Verbose "Testing connection of $c"
         If (Test-Connection -ComputerName $c -Quiet -Count 1) {
             Write-Verbose "Validating path to hosts file"
 
             if ($c -eq "localhost") {
                 $root = "C:"
             } else {
                 $root = "\\$c\C`$"
             }
 
             If (Test-Path "$root\Windows\system32\drivers\etc\hosts") {
                 Switch -regex -file ("$root\Windows\system32\drivers\etc\hosts") {
                     "^#\w+" {
                     }
                     "^\d\w+" {
                         Write-Verbose "Adding IPV4 information to collection"
 
                         $new = $_.Split("") | Where-Object {$_ -ne ""}
                         If ($new[2] -eq $null) {
                             $notes = $null
                         } Else {
                             $notes = $new[2]
                         }
                         New-Object PSObject -Property @{
                             ComputerName = $c
                             IPV4 = $new[0]
                             IPV6 = $null
                             Hostname = $new[1]
                             Notes = $notes
                         }
                     }
                     Default {
                         If (!("\s+" -match $_ -OR $_.StartsWith("#"))) {
                             Write-Verbose "Adding IPV6 information to collection"
 
                             $new = $_.Split("") | ? {$_ -ne ""}
                             If ($new[2] -eq $null) {
                                 $notes = $null
                             } Else {
                                 $notes = $new[2]
                             }
                             New-Object PSObject -Property @{
                                 ComputerName = $c
                                 IPV4 = $null
                                 IPV6 = $new[0]
                                 Hostname = $new[1]
                                 Notes = $notes
                             }
                         }
                     }                        
                 }
             } ElseIf (Test-Path "$root\WinNT\system32\drivers\etc\hosts") {
                 Switch -regex -file ("$root\WinNT\system32\drivers\etc\hosts") {
                     "^#\w+" {
                     }
                     "^\d\w+" {
                         Write-Verbose "Adding IPV4 information to collection"
 
                         $new = $_.Split("") | ? {$_ -ne ""}
                         If ($new[2] -eq $null) {
                             $notes = $null
                         } Else {
                             $notes = $new[2]
                         }
                         New-Object PSObject -Property @{
                             ComputerName = $c
                             IPV4 = $new[0]
                             IPV6 = $null
                             Hostname = $new[1]
                             Notes = $notes
                         }
                     }
                     Default {
                         If (!("\s+" -match $_ -OR $_.StartsWith("#"))) {
                             Write-Verbose "Adding IPV6 information to collection"
 
                             $new = $_.Split("") | ? {$_ -ne ""}
                             If ($new[2] -eq $null) {
                                 $notes = $null
                             } Else {
                                 $notes = $new[2]
                             }
                             New-Object PSObject -Property @{
                                 ComputerName = $c
                                 IPV4 = $null
                                 IPV6 = $new[0]
                                 Hostname = $new[1]
                                 Notes = $notes
                             }
                         }
                     }                        
                 }        
             } Else {
                 Write-Error "Unable to locate host file on computer: $c"
             }
         } Else {
             Write-Error "Unable to locate computer: $c"    
         }
     }
 }
 }
`

