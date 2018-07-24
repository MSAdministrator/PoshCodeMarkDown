---
Author: dollarunderscore
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5032
Published Date: 2014-03-31t19
Archived Date: 2014-05-31t08
---

# amp/receiver control - 

## Description

i will post a module for onkyo receivers when i get the time, this code is only meant as an example on how to figure out how different devices work and how to control them using powershell.

## Comments



## Usage



## TODO



## function

`send-onkyocommand`

## Code

`#
 #
 function Send-ONKYOCommand
 {
     [cmdletbinding()]
     param($OnkyoIP,
           [int] $Port = 60128,
           $Command)
 
 
     BEGIN {
         $Saddrf = [System.Net.Sockets.AddressFamily]::InterNetwork 
         $Stype = [System.Net.Sockets.SocketType]::Stream 
         $Ptype = [System.Net.Sockets.ProtocolType]::TCP
         $Socket = New-Object System.Net.Sockets.Socket $saddrf, $stype, $ptype 
         $Socket.TTL = 32
     }
 
     PROCESS {
 
         $ReceiverAddress = [System.Net.IPAddress]::Parse($OnkyoIP)
 
         $Endpoint = New-Object System.Net.IPEndPoint $ReceiverAddress, $Port
 
         $Socket.Connect($Endpoint)
 
 
         $CommandBytes = $Command.ToCharArray() | % { [BYTE]$_ }
 
         $CommandLength = $Command.Length + 1
 
         $MessageHeader = 73,83,67,80,0,0,0,16,0,0,0,$CommandLength,1,0,0,0
 
         [Byte[]] $Packet = $MessageHeader + $CommandBytes + 0x0D
 
         $Sent = $Socket.Send($Packet)
 
         Write-Verbose "Sent $Sent characters to $OnkyoIP on port $Port. The byte array was: $Packet."
 
         $KeepListening = $true
         $EmptyByteArray = New-Object System.Byte[] 50
 
         do {
             Start-Sleep -Milliseconds 50
 
             $Buffer = New-Object System.Byte[] 50
             
             $Response = $Socket.Receive($Buffer)
 
             if ((Compare-Object $EmptyByteArray $Buffer).length -ne 0) {
                 $ConvertedResponse = ($Buffer | % { if ($_ -ne 0) { [CHAR][BYTE]$_ } }) -join ""
                 Write-Verbose "Response: $ConvertedResponse"
                 $KeepListening = $false
             }
 
             }
         while ($KeepListening)
     }
 
     END {
         $Socket.Close()
         $Socket.Dispose()
     }
 }
`

