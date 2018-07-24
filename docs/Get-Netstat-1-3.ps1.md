---
Author: ivan f
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2694
Published Date: 2011-05-25t15
Archived Date: 2016-06-20t00
---

# get-netstat 1,3 - 

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
 [regex]$regexTCP = '(?<Protocol>\S+)\s+(?<LAddress>\S+):(?<LPort>\S+)\s+(?<RAddress>\S+):(?<RPort>\S+)\s+(?<State>\S+)\s+(?<PID>\S+)'
 [regex]$regexUDP = '(?<Protocol>\S+)\s+(?<LAddress>\S+):(?<LPort>\S+)\s+(?<RAddress>\S+):(?<RPort>\S+)\s+(?<PID>\S+)'
 
 [psobject]$process = "" | Select-Object Protocol, LocalAddress, Localport, RemoteAddress, Remoteport, State, PID, ProcessName
 
 foreach ($net in $netstat)
 {
     switch -regex ($net.Trim())
     {
         $regexTCP
         {      
 	   $process = "" | Select-Object Protocol, LocalAddress, Localport, RemoteAddress, Remoteport, State, PID, ProcessName
             $process.Protocol = $matches.Protocol
             $process.LocalAddress = $matches.LAddress
             $process.Localport = $matches.LPort
             $process.RemoteAddress = $matches.RAddress
             $process.Remoteport = $matches.RPort
             $process.State = $matches.State
             $process.PID = $matches.PID
             $process.ProcessName = ( $ps | Where-Object {$_.Id -eq $matches.PID} ).ProcessName
 	   $process
 	   continue
         }
         $regexUDP
         {         
 	   $process = "" | Select-Object Protocol, LocalAddress, Localport, RemoteAddress, Remoteport, State, PID, ProcessName
             $process.Protocol = $matches.Protocol
             $process.LocalAddress = $matches.LAddress
             $process.Localport = $matches.LPort
             $process.RemoteAddress = $matches.RAddress
             $process.Remoteport = $matches.RPort
             $process.State = $matches.State
             $process.PID = $matches.PID
             $process.ProcessName = ( $ps | Where-Object {$_.Id -eq $matches.PID} ).ProcessName
 	   $process
 	   continue
         }
     }
 }
`

