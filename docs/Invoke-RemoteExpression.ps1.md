---
Author: lee holmes
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 3085
Published Date: 2012-12-09t02
Archived Date: 2016-06-10t16
---

# invoke-remoteexpression. - 

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
 
 Invoke a PowerShell expression on a remote machine. Requires PsExec from
 http://live.sysinternals.com/tools/psexec.exe. If the remote machine
 supports PowerShell version two, use PowerShell remoting instead.
 
 .EXAMPLE
 
 Invoke-RemoteExpression \\LEE-DESK { Get-Process }
 Retrieves the output of a simple command from a remote machine
 
 .EXAMPLE
 
 (Invoke-RemoteExpression \\LEE-DESK { Get-Date }).AddDays(1)
 Invokes a command on a remote machine. Since the command returns one of
 PowerShell's primitive types (a DateTime object,) you can manipulate
 its output as an object afterward.
 
 .EXAMPLE
 
 Invoke-RemoteExpression \\LEE-DESK { Get-Process } | Sort Handles
 Invokes a command on a remote machine. The command does not return one of
 PowerShell's primitive types, but you can still use PowerShell's filtering
 cmdlets to work with its structured output.
 
 #>
 
 param(
     $ComputerName = "\\$ENV:ComputerName",
 
     [Parameter(Mandatory = $true)]
     [ScriptBlock] $ScriptBlock,
 
     $Credential,
 
     [switch] $NoProfile
 )
 
 Set-StrictMode -Version Latest
 
 $commandLine = "echo . | powershell -Output XML "
 
 if($noProfile)
 {
     $commandLine += "-NoProfile "
 }
 
 $commandBytes = [System.Text.Encoding]::Unicode.GetBytes($scriptblock)
 $encodedCommand = [Convert]::ToBase64String($commandBytes)
 $commandLine += "-EncodedCommand $encodedCommand"
 
 $errorOutput = [IO.Path]::GetTempFileName()
 
 if($Credential)
 {
     $credential = Get-Credential $credential
     $networkCredential = $credential.GetNetworkCredential()
     $username = $networkCredential.Username
     $password = $networkCredential.Password
 
     $output = psexec $computername /user $username /password $password `
         /accepteula cmd /c $commandLine 2>$errorOutput
 }
 else
 {
     $output = psexec /acceptEula $computername `
         cmd /c $commandLine 2>$errorOutput
 }
 
 $errorContent = Get-Content $errorOutput
 Remove-Item $errorOutput
 if($errorContent -match "(Access is denied)|(failure)|(Couldn't)")
 {
     $OFS = "`n"
     $errorMessage = "Could not execute remote expression. "
     $errorMessage += "Ensure that your account has administrative " +
         "privileges on the target machine.`n"
     $errorMessage += ($errorContent -match "psexec.exe :")
 
     Write-Error $errorMessage
 }
 
 $output
`

