---
Author: bsonposh
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 85
Published Date: 2008-12-31t14
Archived Date: 2017-05-22t04
---

# test-port.ps1 - 

## Description

test-port creates a tcp connection to specified port. by default it connects to port 135 with a timeout of 3secs.

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 Param([string]$srv,$port=135,$timeout=3000,[switch]$verbose)
  
  
 $ErrorActionPreference = "SilentlyContinue"
  
 $tcpclient = new-Object system.Net.Sockets.TcpClient
  
 $iar = $tcpclient.BeginConnect($srv,$port,$null,$null)
  
 $wait = $iar.AsyncWaitHandle.WaitOne($timeout,$false)
  
 if(!$wait)
 {
     $tcpclient.Close()
     if($verbose){Write-Host "Connection Timeout"}
     Return $false
 }
 else
 {
     $error.Clear()
     $tcpclient.EndConnect($iar) | out-Null
     if(!$?){if($verbose){write-host $error[0]};$failed = $true}
     $tcpclient.Close()
 }
  
 if($failed){return $false}else{return $true}
`

