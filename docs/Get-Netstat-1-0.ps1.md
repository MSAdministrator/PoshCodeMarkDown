---
Author: glnsize
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 884
Published Date: 
Archived Date: 2009-12-11t07
---

# get-netstat 1,0 - 

## Description

it would really be sweet if i could get-netstat -sate close_wait

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 $null, $null, $null, $null, $netstat = netstat -a -n -o
 [regex]$regexTCP = '(?<Protocol>\S+)\s+(?<LocalAddress>\S+)\s+(?<RemoteAddress>\S+)\s+(?<State>\S+)\s+(?<PID>\S+)'
 [regex]$regexUDP = '(?<Protocol>\S+)\s+(?<LocalAddress>\S+)\s+(?<RemoteAddress>\S+)\s+(?<PID>\S+)'
 foreach ($net in $netstat)
 {
     switch -regex ($net.Trim())
     {
         $regexTCP
         {			
             $process = "" | Select-Object Protocol, LocalAddress, RemoteAddress, State, PID, ProcessName
             $process.Protocol = $matches.Protocol
             $process.LocalAddress = $matches.LocalAddress
             $process.RemoteAddress = $matches.RemoteAddress
             $process.State = $matches.State
             $process.PID = $matches.PID
             $process.ProcessName = ( Get-Process -Id $matches.PID ).ProcessName
             $process
             continue
         }
         $regexUDP
         {
             $process = "" | Select-Object Protocol, LocalAddress, RemoteAddress, State, PID, ProcessName
             $process.Protocol = $matches.Protocol
             $process.LocalAddress = $matches.LocalAddress
             $process.PID = $matches.PID
 	   $process.ProcessName = ( Get-Process -Id $matches.PID ).ProcessName
             $process
             continue
         }
     }
 }
`

