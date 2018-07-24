---
Author: lee holmes
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2141
Published Date: 2011-09-09t21
Archived Date: 2016-05-17t09
---

# enable-remotepsremoting. - 

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
 
 Enables PowerShell Remoting on a remote computer. Requires that the machine
 responds to WMI requests, and that its operating system is Windows Vista or
 later.
 
 .EXAMPLE
 
 Enable-RemotePsRemoting <Computer>
 
 #>
 
 param(
     $Computername,
 
     $Credential = (Get-Credential)
 )
 
 Set-StrictMode -Version Latest
 $VerbosePreference = "Continue"
 
 $credential = Get-Credential $credential
 $username = $credential.Username
 $password = $credential.GetNetworkCredential().Password
 
 $script = @"
 
 `$log = Join-Path `$env:TEMP Enable-RemotePsRemoting.output.txt
 Remove-Item -Force `$log -ErrorAction SilentlyContinue
 Start-Transcript -Path `$log
 
 schtasks /CREATE /TN 'Enable Remoting' /SC WEEKLY /RL HIGHEST ``
     /RU $username /RP $password ``
     /TR "powershell -noprofile -command Enable-PsRemoting -Force" /F |
     Out-String
 schtasks /RUN /TN 'Enable Remoting' | Out-String
 
 `$securePass = ConvertTo-SecureString $password -AsPlainText -Force
 `$credential =
     New-Object Management.Automation.PsCredential $username,`$securepass
 
 for(`$count = 1; `$count -le 10; `$count++)
 {
     `$output = Invoke-Command localhost { 1 } -Cred `$credential ``
         -ErrorAction SilentlyContinue
     if(`$output -eq 1) { break; }
 
     "Attempt `$count : Not ready yet."
     Sleep 5
 }
 
 schtasks /DELETE /TN 'Enable Remoting' /F | Out-String
 Stop-Transcript
 
 "@
 
 $commandBytes = [System.Text.Encoding]::Unicode.GetBytes($script)
 $encoded = [Convert]::ToBase64String($commandBytes)
 
 Write-Verbose "Configuring $computername"
 $command = "powershell -NoProfile -EncodedCommand $encoded"
 $null = Invoke-WmiMethod -Computer $computername -Credential $credential `
     Win32_Process Create -Args $command
 
 Write-Verbose "Testing connection"
 Invoke-Command $computername {
     Get-WmiObject Win32_ComputerSystem } -Credential $credential
`

