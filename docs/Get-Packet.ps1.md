---
Author: robbie foust
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 764
Published Date: 2009-12-30t16
Archived Date: 2017-05-03t10
---

# get-packet - 

## Description

this is an updated version of get-packet, an ip packet sniffer for powershell.

## Comments



## Usage

get-packet.ps1 [-localip [<string>]] [-protocol [<string>]] [[-seconds] [<int32>]] [-resolvehosts] [-statistics] [-silent]

## TODO



## script

``

## Code

`#
 #
 #
 #
 #
 #
 #
 #
 #
 
 param([string]$LocalIP = "NotSpecified", [string]$Protocol = "all", [int]$Seconds = 0, [switch]$ResolveHosts, [switch]$Statistics, [switch]$Silent)
 
 $starttime = get-date
 $byteIn = new-object byte[] 4
 $byteOut = new-object byte[] 4
 
 $byteIn[1-3] = 0
 $byteOut[0-3] = 0
 
 $TCPFIN = [byte]0x01
 $TCPSYN = [byte]0x02
 $TCPRST = [byte]0x04
 $TCPPSH = [byte]0x08
 $TCPACK = [byte]0x10
 $TCPURG = [byte]0x20
 
 function NetworkToHostUInt16 ($value)
 	{
 	[Array]::Reverse($value)
 	[BitConverter]::ToUInt16($value,0)
 	}
 
 function NetworkToHostUInt32 ($value)
 	{
 	[Array]::Reverse($value)
 	[BitConverter]::ToUInt32($value,0)
 	}
 
 function ByteToString ($value)
 	{
 	$AsciiEncoding = new-object system.text.asciiencoding
 	$AsciiEncoding.GetString($value)
 	}
 
 
 function ResolveIP ($ip)
 	{
 	if ($data = $hostcache."$($ip.IPAddressToString)")
 		{
 		if ($ip.IPAddressToString -eq $data)
 			{
 			[system.net.ipaddress]$ip
 			}
 		else
 			{
 			$data
 			}
 		}
 	else
 		{
 		$null,$null,$null,$data = nslookup $ip.IPAddressToString 2>$null
 
 		$data = $data -match "Name:"
 
 		if ($data -match "Name:")
 			{
 			$data = $data[0] -replace "Name:\s+",""
 			$hostcache."$($ip.IPAddressToString)" = "$data"
 			$data
 			}
 		else
 			{
 			$hostcache."$($ip.IPAddressToString)" = "$($ip.IPAddressToString)"
 			$ip
 			}
 		}
 	}
 
 if ($LocalIP -eq "NotSpecified") {
 	route print 0* | % { 
 		if ($_ -match "\s{2,}0\.0\.0\.0") { 
 			$null,$null,$null,$LocalIP,$null = [regex]::replace($_.trimstart(" "),"\s{2,}",",").split(",")
 			}
 		}
 	}
 
 write-host "Using IPv4 Address: $LocalIP"
 write-host
 
 
 $socket = new-object system.net.sockets.socket([Net.Sockets.AddressFamily]::InterNetwork,[Net.Sockets.SocketType]::Raw,[Net.Sockets.ProtocolType]::IP)
 
 $socket.setsocketoption("IP","HeaderIncluded",$true)
 
 $socket.ReceiveBufferSize = 819200
 
 $ipendpoint = new-object system.net.ipendpoint([net.ipaddress]"$localIP",0)
 $socket.bind($ipendpoint)
 
 [void]$socket.iocontrol([net.sockets.iocontrolcode]::ReceiveAll,$byteIn,$byteOut)
 
 write-host "Press ESC to stop the packet sniffer ..." -fore yellow
 
 $escKey = 27
 $running = $true
 
 while ($running)
 	{
 	if ($host.ui.RawUi.KeyAvailable)
 		{
 		$key = $host.ui.RawUI.ReadKey("NoEcho,IncludeKeyUp,IncludeKeyDown")
 
 		if ($key.VirtualKeyCode -eq $ESCkey)
 			{
 			$running = $false
 			}
 		}
 	
 		{
 		exit
 		}
 
 		{
 		start-sleep -milliseconds 500
 		continue
 		}
 	
 	$rcv = $socket.receive($byteData,0,$byteData.length,[net.sockets.socketflags]::None)
 
 	$MemoryStream = new-object System.IO.MemoryStream($byteData,0,$rcv)
 	$BinaryReader = new-object System.IO.BinaryReader($MemoryStream)
 
 	$VersionAndHeaderLength = $BinaryReader.ReadByte()
 
 	$TypeOfService= $BinaryReader.ReadByte()
 
 	$TotalLength = NetworkToHostUInt16 $BinaryReader.ReadBytes(2)
 
 	$Identification = NetworkToHostUInt16 $BinaryReader.ReadBytes(2)
 	$FlagsAndOffset = NetworkToHostUInt16 $BinaryReader.ReadBytes(2)
 	$TTL = $BinaryReader.ReadByte()
 	$ProtocolNumber = $BinaryReader.ReadByte()
 	$Checksum = [Net.IPAddress]::NetworkToHostOrder($BinaryReader.ReadInt16())
 
 	$SourceIPAddress = $BinaryReader.ReadUInt32()
 	$SourceIPAddress = [System.Net.IPAddress]$SourceIPAddress
 	$DestinationIPAddress = $BinaryReader.ReadUInt32()
 	$DestinationIPAddress = [System.Net.IPAddress]$DestinationIPAddress
 
 	$ipVersion = [int]"0x$(('{0:X}' -f $VersionAndHeaderLength)[0])"
 
 	$HeaderLength = [int]"0x$(('{0:X}' -f $VersionAndHeaderLength)[1])" * 4
 
 		{
 		}
 	
 	$Data = ""
 	$TCPWindow = ""
 	$SequenceNumber = ""
 	
 		{
 			$protocolDesc = "ICMP"
 
 			$sourcePort = [uint16]0
 			$destPort = [uint16]0
 			break
 			}
 			$protocolDesc = "IGMP"
 			$sourcePort = [uint16]0
 			$destPort = [uint16]0
 			$IGMPType = $BinaryReader.ReadByte()
 			$IGMPMaxRespTime = $BinaryReader.ReadByte()
 			$IGMPChecksum = [System.Net.IPAddress]::NetworkToHostOrder($BinaryReader.ReadInt16())
 			$Data = ByteToString $BinaryReader.ReadBytes($TotalLength - ($HeaderLength - 32))
 			}
 			$protocolDesc = "TCP"
 			
 			$sourcePort = NetworkToHostUInt16 $BinaryReader.ReadBytes(2)
 			$destPort = NetworkToHostUInt16 $BinaryReader.ReadBytes(2)
 			$SequenceNumber = NetworkToHostUInt32 $BinaryReader.ReadBytes(4)
 			$AckNumber = NetworkToHostUInt32 $BinaryReader.ReadBytes(4)
 			
 
 			switch ($TCPFlags)
 				{
 				{ $_ -band $TCPFIN } { $TCPFlagsString += "FIN" }
 				{ $_ -band $TCPSYN } { $TCPFlagsString += "SYN" }
 				{ $_ -band $TCPRST } { $TCPFlagsString += "RST" }
 				{ $_ -band $TCPPSH } { $TCPFlagsString += "PSH" }
 				{ $_ -band $TCPACK } { $TCPFlagsString += "ACK" }
 				{ $_ -band $TCPURG } { $TCPFlagsString += "URG" }
 				}
 			
 			$TCPWindow = NetworkToHostUInt16 $BinaryReader.ReadBytes(2)
 			$TCPChecksum = [System.Net.IPAddress]::NetworkToHostOrder($BinaryReader.ReadInt16())
 			$TCPUrgentPointer = NetworkToHostUInt16 $BinaryReader.ReadBytes(2)
 
 				{
 				[void]$BinaryReader.ReadBytes($TCPHeaderLength - 20)
 				}
 
 			if ($TCPFlags -band $TCPSYN)
 				{
 				$ISN = $SequenceNumber
 				#$SequenceNumber = $BinaryReader.ReadBytes(1)
 				[void]$BinaryReader.ReadBytes(1)
 				}
 
 			$Data = ByteToString $BinaryReader.ReadBytes($TotalLength - ($HeaderLength + $TCPHeaderLength))
 			break
 			}
 			$protocolDesc = "UDP"
 
 			$sourcePort = NetworkToHostUInt16 $BinaryReader.ReadBytes(2)
 			$destPort = NetworkToHostUInt16 $BinaryReader.ReadBytes(2)
 			$UDPLength = NetworkToHostUInt16 $BinaryReader.ReadBytes(2)
 			[void]$BinaryReader.ReadBytes(2)
 			$Data = ByteToString $BinaryReader.ReadBytes(($UDPLength - 2) * 4)
 			break
 			}
 		default {
 			$protocolDesc = "Other ($_)"
 			$sourcePort = 0
 			$destPort = 0
 			break
 			}
 		}
 	
 	$BinaryReader.Close()
 	$memorystream.Close()
 
 		{
 
 		$DestinationHostName = ResolveIP($DestinationIPAddress)
 		$SourceHostName = ResolveIP($SourceIPAddress)
 		}
 
 
 	if ($Protocol -eq "all" -or $Protocol -eq $protocolDesc)
 		{
 		$packet = new-object psobject
 
 		$packet | add-member noteproperty Destination $DestinationIPAddress
 		if ($ResolveHosts) { $packet | add-member noteproperty DestinationHostName $DestinationHostName }
 		$packet | add-member noteproperty Source $SourceIPAddress
 		if ($ResolveHosts) { $packet | add-member noteproperty SourceHostName $SourceHostName }
 		$packet | add-member noteproperty Version $ipVersion
 		$packet | add-member noteproperty Protocol $protocolDesc
 		$packet | add-member noteproperty Sequence $SequenceNumber
 		$packet | add-member noteproperty Window $TCPWindow
 		$packet | add-member noteproperty DestPort $destPort
 		$packet | add-member noteproperty SourcePort $sourcePort
 		$packet | add-member noteproperty Flags $TCPFlagsString
 		$packet | add-member noteproperty Data $Data
 		$packet | add-member noteproperty Time (get-date)
 
 
 		if (-not $Silent)
 			{
 			$packet
 			}
 		}
 	}
 
 if ($Statistics)
 	{
 	$activity = "Analyzing network trace"
 
 	write-progress $activity "Counting packets"
 	$elapsed = $packets[-1].time - $packets[0].time
 
 	write-progress $activity "Calculating elapsed time"
 	$pps = $packets.count/(($packets[-1].time -$packets[0].time).totalseconds)
 	$pps="{0:N4}" -f $pps
 
 	write-progress $activity "Calculating protocol distribution"
 	$protocols = $packets | sort protocol | group protocol | sort count -descending | select Count,@{name="Protocol";Expression={$_.name}} 
 
 	write-progress $activity "Calculating source port distribution"
 	$sourceport = $packets | sort sourceport | group sourceport | sort count -descending | select Count,@{name="Port";Expression={$_.name}}
 
 	write-progress $activity "Calculating destination distribution"
 	$destinationlist = $packets | sort Destination | select Destination
 
 	write-progress $activity "Calculating destination port distribution"
 	$destinationport = $packets | sort destport | group destport | sort count -descending | select Count,@{name="Port";Expression={$_.name}}
 
 	write-progress $activity "Building source list"
 	$sourcelist = $packets | sort source | select Source
 
 	write-progress $activity "Building source IP list"
 	$ips = $sourcelist | group source | sort count -descending | select Count,@{Name="IP";Expression={$_.Name}}
 		
 	write-progress $activity "Building destination IP list"
 	$ipd = $destinationlist | group destination | sort count -descending | select Count,@{Name="IP";Expression={$_.Name}}
 
 	write-progress $activity "Compiling results"
 	$protocols = $protocols | Select Count,Protocol,@{Name="Percentage";Expression={"{0:P4}" -f ($_.count/$packets.count)}} 
 
 	$destinationport = $destinationport | select Count,Port,@{Name="Percentage";Expression={"{0:P4}" -f ($_.count/$packets.count)}} 
 
 	$sourceport = $sourceport | Select Count,Port,@{Name="Percentage";Expression={"{0:P4}" -f ($_.count/$packets.count)}} 
 
 	if ($ResolveHosts)
 		{
 		write-progress $activity "Resolving IPs"
 
 		foreach ($destination in $ipd)
 			{
 			$destination | add-member noteproperty "Host" $(ResolveIP([system.net.ipaddress]$destination.IP))
 			}
 		foreach ($source in $ips)
 			{
 			$source | add-member noteproperty "Host" $(ResolveIP([system.net.ipaddress]$source.IP))
 			}
 		}
 
 	write-progress $activity "Compiling results"
 	$destinations = $ipd | Select Count,IP,Host,@{Name="Percentage";Expression={"{0:P4}" -f ($_.count/$packets.count)}} 
 	$sources = $ips | Select Count,IP,Host,@{Name="Percentage";Expression={"{0:P4}" -f ($_.count/$packets.count)}} 
 
 	$global:stats = new-object psobject
 
 	$stats | add-member noteproperty "TotalPackets" $packets.count
 	$stats | add-member noteproperty "Elapsedtime" $elapsed
 	$stats | add-member noteproperty "PacketsPerSec" $pps
 	$stats | add-member noteproperty "Protocols" $protocols
 	$stats | add-member noteproperty "Destinations" $destinations
 	$stats | add-member noteproperty "DestinationPorts" $destinationport
 	$stats | add-member noteproperty "Sources" $sources
 	$stats | add-member noteproperty "SourcePorts" $sourceport 
 
 	write-host
 	write-host " TotalPackets: " $stats.totalpackets
 	write-host "  ElapsedTime: " $stats.elapsedtime
 	write-host "PacketsPerSec: " $stats.packetspersec
 	write-host
 	write-host "More statistics can be accessed from the global `$stats variable." -fore cyan
 	
 	}
`

