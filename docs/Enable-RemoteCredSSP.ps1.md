---
Author: lee holmes
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2140
Published Date: 2011-09-09t21
Archived Date: 2017-02-16t15
---

# enable-remotecredssp.ps1 - 

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
 
 Enables CredSSP support on a remote computer. Requires that the machine
 have PowerShell Remoting enabled, and that its operating system is Windows
 Vista or later.
 
 .EXAMPLE
 
 Enable-RemoteCredSSP <Computer>
 
 #>
 
 param(
     $Computername,
 
     $Credential = (Get-Credential)
 )
 
 Set-StrictMode -Version Latest
 
 $credential = Get-Credential $credential
 $username = $credential.Username
 $password = $credential.GetNetworkCredential().Password
 
 $powerShellCommand =
     "powershell -noprofile -command Enable-WsManCredSSP -Role Server -Force"
 $script = @"
 schtasks /CREATE /TN 'Enable CredSSP' /SC WEEKLY /RL HIGHEST ``
     /RU $username /RP $password ``
     /TR "$powerShellCommand" /F
 
 schtasks /RUN /TN 'Enable CredSSP'
 "@
 
 $command = [ScriptBlock]::Create($script)
 Invoke-Command $computername $command -Cred $credential
 
 for($count = 1; $count -le 10; $count++)
 {
     $output =
         Invoke-Command $computername { 1 } -Auth CredSSP -Cred $credential
     if($output -eq 1) { break; }
 
     "Attempt $count : Not ready yet."
     Sleep 5
 }
 
 $command = [ScriptBlock]::Create($script)
 Invoke-Command $computername {
     schtasks /DELETE /TN 'Enable CredSSP' /F } -Cred $credential
 
 Invoke-Command $computername {
     Get-WmiObject Win32_ComputerSystem } -Auth CredSSP -Cred $credential
`

