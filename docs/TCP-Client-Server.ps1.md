---
Author: rtsssdada
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6759
Published Date: 2017-02-27t21
Archived Date: 2017-03-31t03
---

# tcp client/server - 

## Description

an example of a client/server that works in powershell

## Comments



## Usage



## TODO



## function

`listen-port`

## Code

`#
 #
 function listen-port ($port=8989) {
     $endpoint = new-object System.Net.IPEndPoint ([system.net.ipaddress]::any, $port)
     $listener = new-object System.Net.Sockets.TcpListener $endpoint
     $listener.start()
 
     do {
         $stream = $client.GetStream();
         $reader = New-Object System.IO.StreamReader $stream
         do {
 
             $line = $reader.ReadLine()
             write-host $line -fore cyan
         } while ($line -and $line -ne ([char]4))
         $reader.Dispose()
         $stream.Dispose()
         $client.Dispose()
     } while ($line -ne ([char]4))
     $listener.stop()
 }
 
 
 
 function send-msg ($message=$([char]4), $port=8989, $server="localhost") {
     $client = New-Object System.Net.Sockets.TcpClient $server, $port
     $stream = $client.GetStream()
     $writer = New-Object System.IO.StreamWriter $stream
     $writer.Write($message)
     $writer.Close()
     $stream.Dispose()
     $client.Close()
 }
`

