---
Author: yavu305z
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5829
Published Date: 2015-04-16t08
Archived Date: 2015-04-19t06
---

# scriptable telnet client - 

## Description

i wrote this a while back to be able to automate stuff on a hp san and ilo. it’s pretty much mritten ad-hoc so please feel free to improve on it. anyway, might be useful for someone out there. the function send-command logs on to a telnet server and executes the piped in commands.

## Comments



## Usage



## TODO



## function

`read-stream`

## Code

`#
 #
 function read-stream ([Parameter(Posistion=0,Mandatory=$true)][validatenotnull()]
 		[System.Net.Sockets.NetworkStream]$stream,
 		[String]$expect = "")
 {
 	$buffer = new-object system.byte[] 1024
 	$enc = new-object system.text.asciiEncoding
 
 
 	start-sleep -m 100
 	$output = ""
 	while($stream.DataAvailable -or $output -notmatch $expect)
 	{   
 		$read = $stream.Read($buffer, 0, 1024)    
 		$output = "$output$($enc.GetString($buffer, 0, $read))"
 		start-sleep -m 100
 	}
 	$output.split("`n")
 }
 
 function send-command ([parameter(position=0,Mandatory=$true)][validatenotnull()]
 		[String]$hostname,
 	[parameter(position=1,Mandatory=$true)][validatenotnull()]
 		[String]$User,
 	[parameter(position=2,Mandatory=$true)][validatenotnull()]
 		[String]$Password, 
 	[parameter(position=3,Mandatory=$true,valuefrompipeline=$true)][validatenotnull()]
 		[String]$InputObject,
 		[string]$Expect = "")
 {
 	begin
 	{
 		
 		$sock = new-object system.net.sockets.tcpclient($hostname,23)
 		$str = $sock.GetStream()
 		$wrt = new-object system.io.streamwriter($str)
 		
 		read-stream($str)
 		$wrt.writeline($user)
 		$wrt.flush()
 		read-stream($str)
 		$wrt.writeline($password)
 		$wrt.flush()
 		read-stream($str, $expect)
 	}
 	process
 	{
 		$wrt.writeline($InputObject)
 		$wrt.flush()
 		read-stream($str, $expect)
 	}
 	end
 	{
 		$wrt.writeline("exit")
 		$wrt.flush()
 		read-stream($str)
 
 		$wrt.Close()
 		$str.Close()
 		$sock.close()
 	}
 }
`

