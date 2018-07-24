---
Author: glnsize
Publisher: 
Copyright: 
Email: 
Version: 1.2
Encoding: ascii
License: cc0
PoshCode ID: 3139
Published Date: 2012-01-03t08
Archived Date: 2016-06-10t16
---

# get-netstat - 

## Description

this will perform a basic netstat.exe command and “objectize” its output.

## Comments



## Usage



## TODO



## function

`get-netstat`

## Code

`#
 #
 Function Get-Netstat {
     $null, $null, $null, $null, $netstat = netstat -a -n -o
     [regex]$regexTCP = '(?<Protocol>\S+)\s+((?<LAddress>(2[0-4]\d|25[0-5]|[01]?\d\d?)\.(2[0-4]\d|25[0-5]|[01]?\d\d?)\.(2[0-4]\d|25[0-5]|[01]?\d\d?)\.(2[0-4]\d|25[0-5]|[01]?\d\d?))|(?<LAddress>\[?[0-9a-fA-f]{0,4}(\:([0-9a-fA-f]{0,4})){1,7}\%?\d?\]))\:(?<Lport>\d+)\s+((?<Raddress>(2[0-4]\d|25[0-5]|[01]?\d\d?)\.(2[0-4]\d|25[0-5]|[01]?\d\d?)\.(2[0-4]\d|25[0-5]|[01]?\d\d?)\.(2[0-4]\d|25[0-5]|[01]?\d\d?))|(?<RAddress>\[?[0-9a-fA-f]{0,4}(\:([0-9a-fA-f]{0,4})){1,7}\%?\d?\]))\:(?<RPort>\d+)\s+(?<State>\w+)\s+(?<PID>\d+$)'
 
     [regex]$regexUDP = '(?<Protocol>\S+)\s+((?<LAddress>(2[0-4]\d|25[0-5]|[01]?\d\d?)\.(2[0-4]\d|25[0-5]|[01]?\d\d?)\.(2[0-4]\d|25[0-5]|[01]?\d\d?)\.(2[0-4]\d|25[0-5]|[01]?\d\d?))|(?<LAddress>\[?[0-9a-fA-f]{0,4}(\:([0-9a-fA-f]{0,4})){1,7}\%?\d?\]))\:(?<Lport>\d+)\s+(?<RAddress>\*)\:(?<RPort>\*)\s+(?<PID>\d+)'
 
     [psobject]$process = "" | Select-Object Protocol, LocalAddress, Localport, RemoteAddress, Remoteport, State, PID, ProcessName, Services
 
     $Services = @{}
     get-wmiobject win32_service | ForEach-Object { 
         [String]$SvcPID = $_.processid
         If ($Services.ContainsKey($SvcPID))
         {
             $Services.Item($SvcPID) = $Services.Item($SvcPID) += $($_.Name)
         }
         Else
         {
             $Services.Add($SvcPID,@($_.Name))
         }
     }
 
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
                 $process.ProcessName = ( Get-Process -Id $matches.PID -ea 0).ProcessName
                 $process.Services = $Services.Item($matches.PID)
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
                 $process.ProcessName = ( Get-Process -Id $matches.PID -ea 0).ProcessName
                 $process.Services = $Services.Item($matches.PID)
             }
         }
     $process
     }
 }
`

