---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 1315
Published Date: 
Archived Date: 2009-09-16t14
---

# test-host v1 - 

## Description

test-host for v1

## Comments



## Usage



## TODO



## function

``

## Code

`#
 #
 Param($property,$tport=135,$timeout=1000,[switch]$port,[switch]$verbose)
 Begin{
     function TestPort {
         Param($srv)
         $error.Clear()
         $ErrorActionPreference = "SilentlyContinue"
         $tcpclient = new-Object system.Net.Sockets.TcpClient
         $iar = $tcpclient.BeginConnect($srv,$tport,$null,$null)
         $wait = $iar.AsyncWaitHandle.WaitOne($timeout,$false)
         trap {if($verbose){Write-Host "General Exception"};return $false}
         trap [System.Net.Sockets.SocketException]
         {
             if($verbose){Write-Host "Exception: $($_.exception.message)"}
             return $false
         }
         if(!$wait)
         {
             $tcpclient.Close()
             if($verbose){Write-Host "Connection Timeout"}
             return $false
         }
         else
         {
             $tcpclient.EndConnect($iar) | out-Null
             $tcpclient.Close()
         }
         if(!$error[0]){return $true}
     }
     function PingServer {
         Param($MyHost)
         $pingresult = Get-WmiObject win32_pingstatus -f "address='$MyHost'"
         if($pingresult.statuscode -eq 0) {$true} else {$false}
     }
 }
 Process{
     if($_)
     {
         if($port)
         {
             if($property)
             {
                 if(TestPort $_.$property){$_}  
             }
             else
             {
                 if(TestPort $_){$_} 
             }
         }
         else
         {
             if($property)
             {
                 if(PingServer $_.$property){$_}  
             }
             else
             {
                 if(PingServer $_){$_}
             }
         }
     }
 }
`

