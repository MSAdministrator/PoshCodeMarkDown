---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 0.3
Encoding: ascii
License: cc0
PoshCode ID: 5557
Published Date: 2016-10-31t07
Archived Date: 2016-09-18t18
---

# whois - 

## Description

a whois script for powershell.

## Comments



## Usage



## TODO



## function

`get-whois`

## Code

`#
 #
     #.Synopsis
     #.Example
     #
     #.Example
     #
     #.Example
     #
     #.Example
     #
     #
     #.Example
     #.Notes
     [CmdletBinding()]
     param(
         [Parameter(Position=0, ValueFromRemainingArguments=$true)]
         [string]$query,
 
         [string]$server,
 
         [switch]$NoForward
     )
     end {
         $TLDs = DATA {
           @{
             ".br.com"="whois.centralnic.net"
             ".cn.com"="whois.centralnic.net"
             ".eu.org"="whois.eu.org"
             ".com"="whois.crsnic.net"
             ".net"="whois.crsnic.net"
             ".org"="whois.publicinterestregistry.net"
             ".edu"="whois.educause.net"
             ".gov"="whois.nic.gov"
           }
         }
 
         $EAP, $ErrorActionPreference = $ErrorActionPreference, "Stop"
 
         $query = $query.Trim()
 
         if($query -match "(?:\d{1,3}\.){3}\d{1,3}") {
             Write-Verbose "IP Lookup!"
             if($query -notmatch " ") {
                 $query = "n $query"
             }
             if(!$server) { $server = "whois.arin.net" }
         } elseif(!$server) {
             $server = $TLDs.GetEnumerator() |
                 Where { $query -like  ("*"+$_.name) } |
                 Select -Expand Value -First 1
         }
 
         if(!$server) { $server = "whois.arin.net" }
         $maxRequery = 3 
 
         do {
             Write-Verbose "Connecting to $server"
             $client = New-Object System.Net.Sockets.TcpClient $server, 43
 
             try {
                 $stream = $client.GetStream()
 
                 Write-Verbose "Sending Query: $query"
                 $data = [System.Text.Encoding]::Ascii.GetBytes( $query + "`r`n" )
                 $stream.Write($data, 0, $data.Length)
 
                 Write-Verbose "Reading Response:"
                 $reader = New-Object System.IO.StreamReader $stream, [System.Text.Encoding]::ASCII
 
                 $result = $reader.ReadToEnd()
 
                 if($result -match "(?s)Whois Server:\s*(\S+)\s*") {
                     Write-Warning "Recommended WHOIS server: ${server}"
                     if(!$NoForward) {
                         Write-verbose "Non-Authoritative Results:`n${result}"
                         if(!$cachedResult) {
                             $cachedResult = $result
                             $cachedServer = $server
                         }
                         $server = $matches[1]
                         $query = ($query -split " ")[-1]
                         $maxRequery--
                     } else { $maxRequery = 0 }
                 } else { $maxRequery = 0 }
             } finally {
                 if($stream) {
                     $stream.Close()
                     $stream.Dispose()
                 }
             }
         } while ($maxRequery -gt 0)
 
         $result
 
         if($cachedResult -and ($result -split "`n").count -lt 5) {
             Write-Warning "Original Result from ${cachedServer}:"
             $cachedResult
         }
 
         $ErrorActionPreference = $EAP
     }
`

