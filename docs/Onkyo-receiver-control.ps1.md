---
Author: parul jain paruljain
Publisher: 
Copyright: 
Email: 
Version: 1.0
Encoding: ascii
License: cc0
PoshCode ID: 5163
Published Date: 2014-05-16t23
Archived Date: 2014-05-20t03
---

# onkyo receiver control - 

## Description

library to discover onkyo network connected home theatre receiver and then send remote control commands to change settings such as volume but also query current settings. uses the integra serial communication protocol (iscp).

## Comments



## Usage



## TODO



## script

`convert-tohex`

## Code

`#
 #
 <#
 .SYNOPSIS
     Onkyo home theatre receiver controller
 .DESCRIPTION
     Library to discover Onkyo network connected home theatre receiver and then send remote control commands to change
     settings such as volume but also query current settings. Uses the Integra Serial Communication Protocol (ISCP).
     Tested with Onkyo TX-NR509. As of now this can only work with one receiver at a time however extending to multiple
     receivers should not be that hard. Please contact the author if you have need for multiple receivers or any other requirements.
     
     Complete Integra Serial Communication Protocol is documented here http://blog.siewert.net/files/ISCP%20AV%20Receiver%20v124-1.xls
 
     Combine this module with a simple PowerShell RESTful server http://poshcode.org/4073 and some HTML/Javascript and you can quickly
     build a nice HTML interface to your Onkyo receiver that can be used from any device including PC, Mac and mobile devices. Very
     useful when you want advanced commands such as dynamic range control that are not available in the official app
 .NOTES
     Author: Parul Jain paruljain@hotmail.com
     Version: 1.0, May 16th, 2014
     Prerequisites: PowerShell v2.0 or higher
 .EXAMPLE
     $receiver = New-OnkyoReceiver
     for ($i=0; $i -lt 6; $i++) { if ($receiver.Discover()) { break } }
     if (!$receiver.connected) { throw 'Unable to discover and connect to receiver' }
     if ($receiver.Send('mvl') -match 'MVL(\d+)') { 'The current volume is ' + ($matches[1] | Convert-ToDec) }
     $receiver.Disconnect()
 #>
 
 function Convert-ToHex {
     Process {
         "{0:X2}" -f $_
     }
 }
 
 function Convert-ToDec {
     Process {
         [Convert]::ToInt32($_, 16).ToString()
     }
 }
 
 function New-OnkyoReceiver {
     $receiver = New-Object PSObject
     $receiver | Add-Member -MemberType NoteProperty -Name Name -Value ''
     $receiver | Add-Member -MemberType NoteProperty -Name IPAddress -Value ''
     $receiver | Add-Member -MemberType NoteProperty -Name Port -Value 60128
     $receiver | Add-Member -MemberType NoteProperty -Name Connected -Value $false
     $receiver | Add-Member -MemberType NoteProperty -Name Stream -Value $null
 
     $receiver | Add-Member -MemberType ScriptMethod -Name Discover -Value {
         $udpClient = New-Object System.Net.Sockets.UdpClient
         $udpClient.EnableBroadcast = $true
         $udpClient.Client.ExclusiveAddressUse = $false
         $ep = New-Object Net.IPEndPoint($([Net.IPAddress]::Any, 0))
         [byte[]]$packet = (73,83,67,80,0,0,0,16,0,0,0,11,1,0,0,0,33,120,69,67,78,81,83,84,78,13,10)
         $udpClient.Send($packet, $packet.Length, '255.255.255.255', 60128) | out-null
         $udpClient.Send($packet, $packet.Length, '255.255.255.255', 60128) | out-null
 
         
         try {
             $response = $udpClient.Receive([Ref]$ep)
             $cmdLength = [BitConverter]::ToInt32($response[11..8], 0)
 
             $this.Name = [System.Text.Encoding]::ASCII.GetString($response[24..(13 + $cmdLength)]).split('/')[0]
             $this.IPAddress = $ep.Address.IPAddressToString
             $this.Port = $ep.Port
             $this.Connect()
         } catch { $false }
     }
 
     $receiver | Add-Member -MemberType ScriptMethod -Name Connect -Value {
         if ($this.Connected) { $true; return }
         try { $socket = New-Object System.Net.Sockets.TcpClient($this.IPAddress, $this.Port) }
         $this.Connected = $true
         $this.Stream = $socket.GetStream()
         $this.Stream.ReadTimeout = 1000
         $this.Stream.WriteTimeout = 1000
         $true
     }
 
     $receiver | Add-Member -MemberType ScriptMethod -Name Disconnect -Value {
         $this.Stream.Close()
         $this.Connected = $false
     }
 
     $receiver | Add-Member -MemberType ScriptMethod -Name Send -Value {
         param ([string]$command)
       
         if (!$this.Connected) { 'Not connected to a receiver'; return }
         if ($command.length -lt 3) { 'Command cannot be empty or less than 3 characters'; return }
         if ($command.Length -eq 3) { $command += 'QSTN' }
 
         [byte[]]$packet = (73,83,67,80,0,0,0,16) + $cmdLength + (1,0,0,0,33,49) + $cmd + (13,10)
         $this.Stream.Write($packet, 0, $packet.Length)
 
         $this.Stream.Flush()
 
         Start-Sleep -Seconds 1
 
         while ($this.Stream.DataAvailable) {
 
             [byte[]]$header = @()
             for ($i = 0; $i -lt 16; $i++) {
                 try { $header += $this.Stream.ReadByte() } catch { 'Response timed out'; return }
             }
     
             $cmdLength = [BitConverter]::ToInt32($header[11..8], 0)
     
             [byte[]]$cmd = @()
             for ($i = 0; $i -lt $cmdLength; $i++) {
                 try { $cmd += $this.Stream.ReadByte() } catch { 'Error reading response'; return }
             }
 
             $response = [System.Text.Encoding]::ASCII.GetString($cmd[2..($cmd.Length - 4)])
         }
         $response
     }
 
     $receiver
 }
 
     $receiver = New-OnkyoReceiver
     for ($i=0; $i -lt 6; $i++) { if ($receiver.Discover()) { break } }
     if (!$receiver.connected) { throw 'Unable to discover and connect to receiver' }
     if ($receiver.Send('mvl') -match 'MVL(\d+)') { 'The current volume is ' + ($matches[1] | Convert-ToDec) }
     $receiver.Disconnect()
 #>
`

