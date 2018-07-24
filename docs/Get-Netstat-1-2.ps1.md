---
Author: glnsize
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2398
Published Date: 2011-12-04t17
Archived Date: 2016-05-28t14
---

# get-netstat 1,2 - 

## Description

this will perform a basic netstat.exe command and “objectize” its output.

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 $null, $null, $null, $null, $netstat = netstat -a -n -o
 $ps = Get-Process
 [regex]$regexTCP = '(?<Protocol>\S+)\s+((?<LAddress>(2[0-4]\d|25[0-5]|[01]?\d\d?)\.(2[0-4]\d|25[0-5]|[01]?\d\d?)\.(2[0-4]\d|25[0-5]|[01]?\d\d?)\.(2[0-4]\d|25[0-5]|[01]?\d\d?))|(?<LAddress>\[?[0-9a-fA-f]{0,4}(\:([0-9a-fA-f]{0,4})){1,7}\%?\d?\]))\:(?<Lport>\d+)\s+((?<Raddress>(2[0-4]\d|25[0-5]|[01]?\d\d?)\.(2[0-4]\d|25[0-5]|[01]?\d\d?)\.(2[0-4]\d|25[0-5]|[01]?\d\d?)\.(2[0-4]\d|25[0-5]|[01]?\d\d?))|(?<RAddress>\[?[0-9a-fA-f]{0,4}(\:([0-9a-fA-f]{0,4})){1,7}\%?\d?\]))\:(?<RPort>\d+)\s+(?<State>\w+)\s+(?<PID>\d+$)'
 
 [regex]$regexUDP = '(?<Protocol>\S+)\s+((?<LAddress>(2[0-4]\d|25[0-5]|[01]?\d\d?)\.(2[0-4]\d|25[0-5]|[01]?\d\d?)\.(2[0-4]\d|25[0-5]|[01]?\d\d?)\.(2[0-4]\d|25[0-5]|[01]?\d\d?))|(?<LAddress>\[?[0-9a-fA-f]{0,4}(\:([0-9a-fA-f]{0,4})){1,7}\%?\d?\]))\:(?<Lport>\d+)\s+(?<RAddress>\*)\:(?<RPort>\*)\s+(?<PID>\d+)'
 
 [psobject]$process = "" | Select-Object Protocol, LocalAddress, Localport, RemoteAddress, Remoteport, State, PID, ProcessName
 
 foreach ($net in $netstat)
 {
     switch -regex ($net.Trim())
     {
         $regexTCP
         {          
             $process.Protocol = $matches.Protocol
             $process.LocalAddress = $matches.LAddress
             $process.Localport = $matches.LPort
             $process.RemoteAddress = $matches.RAddress
             $process.Remoteport = $matches.RPort
             $process.State = $matches.State
             $process.PID = $matches.PID
             $process.ProcessName = ( $ps | Where-Object {$_.Id -eq $matches.PID} ).ProcessName
         }
         $regexUDP
         {          
             $process.Protocol = $matches.Protocol
             $process.LocalAddress = $matches.LAddress
             $process.Localport = $matches.LPort
             $process.RemoteAddress = $matches.RAddress
             $process.Remoteport = $matches.RPort
             $process.State = $matches.State
             $process.PID = $matches.PID
             $process.ProcessName = ( $ps | ? {$_.Id -eq $matches.PID} ).ProcessName
         }
     }
 $process
 }
`

