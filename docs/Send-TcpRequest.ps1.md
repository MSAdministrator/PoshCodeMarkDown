---
Author: lee holmes
Publisher: 
Copyright: 
Email: 
Version: 1.1
Encoding: ascii
License: cc0
PoshCode ID: 2218
Published Date: 2011-09-09t21
Archived Date: 2017-03-01t13
---

# send-tcprequest.ps1 - 

## Description

from windows powershell cookbook (o’reilly) by lee holmes

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 ##############################################################################
 ##
 ##
 ##
 ##############################################################################
 
 <#
 
 .SYNOPSIS
 
 Send a TCP request to a remote computer, and return the response.
 If you do not supply input to this script (via either the pipeline, or the
 -InputObject parameter,) the script operates in interactive mode.
 
 .EXAMPLE
 
 PS >$http = @"
   GET / HTTP/1.1
   Host:bing.com
   `n`n
 "@
 
 $http | Send-TcpRequest bing.com 80
 
 #>
 
 param(
     [string] $ComputerName = "localhost",
 
     [switch] $Test,
 
     [int] $Port = 80,
 
     [switch] $UseSSL,
 
     [string] $InputObject,
 
     [int] $Delay = 100
 )
 
 Set-StrictMode -Version Latest
 
 [string] $SCRIPT:output = ""
 
 $currentInput = $inputObject
 if(-not $currentInput)
 {
     $currentInput = @($input)
 }
 $scriptedMode = ([bool] $currentInput) -or $test
 
 function Main
 {
     if(-not $scriptedMode)
     {
         write-host "Connecting to $computerName on port $port"
     }
 
     try
     {
         $socket = New-Object Net.Sockets.TcpClient($computerName, $port)
     }
     catch
     {
         if($test) { $false }
         else { Write-Error "Could not connect to remote computer: $_" }
 
         return
     }
 
     if($test) { $true; return }
 
     if(-not $scriptedMode)
     {
         write-host "Connected.  Press ^D followed by [ENTER] to exit.`n"
     }
 
     $stream = $socket.GetStream()
 
     if($UseSSL)
     {
         $sslStream = New-Object System.Net.Security.SslStream $stream,$false
         $sslStream.AuthenticateAsClient($computerName)
         $stream = $sslStream
     }
 
     $writer = new-object System.IO.StreamWriter $stream
 
     while($true)
     {
         $SCRIPT:output += GetOutput
 
         if($scriptedMode)
         {
             foreach($line in $currentInput)
             {
                 $writer.WriteLine($line)
                 $writer.Flush()
                 Start-Sleep -m $Delay
                 $SCRIPT:output += GetOutput
             }
 
             break
         }
         else
         {
             if($output)
             {
                 foreach($line in $output.Split("`n"))
                 {
                     write-host $line
                 }
                 $SCRIPT:output = ""
             }
 
             $command = read-host
             if($command -eq ([char] 4)) { break; }
 
             $writer.WriteLine($command)
             $writer.Flush()
         }
     }
 
     $writer.Close()
     $stream.Close()
 
     if($scriptedMode)
     {
         $output
     }
 }
 
 function GetOutput
 {
     $buffer = new-object System.Byte[] 1024
     $encoding = new-object System.Text.AsciiEncoding
 
     $outputBuffer = ""
     $foundMore = $false
 
     do
     {
         start-sleep -m 1000
 
         $foundmore = $false
         $stream.ReadTimeout = 1000
 
         do
         {
             try
             {
                 $read = $stream.Read($buffer, 0, 1024)
 
                 if($read -gt 0)
                 {
                     $foundmore = $true
                     $outputBuffer += ($encoding.GetString($buffer, 0, $read))
                 }
             } catch { $foundMore = $false; $read = 0 }
         } while($read -gt 0)
     } while($foundmore)
 
     $outputBuffer
 }
 
 . Main
`

