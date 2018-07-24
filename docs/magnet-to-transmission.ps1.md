---
Author: meekmaak
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: utf-8
License: cc0
PoshCode ID: 6361
Published Date: 2016-05-28t12
Archived Date: 2016-05-30t23
---

# magnet to transmission - 

## Description

the following script shows how to send a magnet link to a remote transmission daemon. don’t forget to replace the $addr, $port, $user and $pass variables at the top before using it.

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 [CmdletBinding()]
 Param(
    [Parameter(Mandatory=$True)]
    [string]$magnetlink
 )
 
 #--------------------------------------------------------------------------
 $addr="192.168.178.25"
 $port="8111"
 $user="admin"
 $pass="admin"
 #--------------------------------------------------------------------------
 
 $uri="http://$($addr):$($port)/transmission/rpc"
 $PWord = ConvertTo-SecureString �String $pass �AsPlainText -Force
 
 $cred = New-Object �TypeName System.Management.Automation.PSCredential �ArgumentList $user, $PWord
 
 try
 {
 	Invoke-RestMethod -Uri $uri -Credential $cred -ErrorAction SilentlyContinue
 }
 catch
 {
 }
 
 $response=$error[0].ErrorDetails.Message
 $sessionid=$response | Select-String -pattern 'X-Transmission-Session-Id\: (?<sessid>[a-zA-Z0-9]*)' | Foreach {$_.Matches.Groups[1].Value } 
 
 $body = "{`"method`":`"torrent-add`",`"arguments`":{`"filename`":`"$magnetlink`"}}"
 
 Invoke-RestMethod -Uri $uri -Headers @{"X-Transmission-Session-Id"=$sessionid} -Credential $cred -Method Post -Body $body
`

