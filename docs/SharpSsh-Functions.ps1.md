---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 877
Published Date: 2009-02-17t09
Archived Date: 2015-04-25t03
---

# sharpssh functions - 

## Description

a few wrapper functions to make working with the ssh portion of sharpssh easier

## Comments



## Usage



## TODO



## function

`new-sshsession`

## Code

`#
 #
 [void][reflection.assembly]::LoadFrom( (Resolve-Path "~\Documents\WindowsPowerShell\Libraries\Tamir.SharpSSH.dll") )
 
 ##
 
 function New-SshSession {
 Param(
    [string]$UserName
 ,  [string]$HostName
 ,  [string]$RSAKeyFile
 ,  [switch]$Passthru
 )
    if($RSAKeyFile -and (Test-Path $RSAKeyFile)){
       $global:LastSshSession = new-object Tamir.SharpSsh.SshShell `
                                           $cred.GetNetworkCredential().Domain, 
                                           $cred.GetNetworkCredential().UserName
       $global:LastSshSession.AddIdentityFile( (Resolve-Path $RSAKeyFile) )
    }
    else {
       $cred = $host.UI.PromptForCredential("SSH Login Credentials",
                                            "Please specify credentials in user@host format",
                                            "$UserName@$HostName","")
       $global:LastSshSession = new-object Tamir.SharpSsh.SshShell `
                                           $cred.GetNetworkCredential().Domain, 
                                           $cred.GetNetworkCredential().UserName,
                                           $cred.GetNetworkCredential().Password
    }
 
 
    $global:LastSshSession.Connect()
    $global:LastSshSession.RemoveTerminalEmulationCharacters = $true
    if($Passthru) {
       return $global:LastSshSession
    }
 }
 
 function Remove-SshSession {
 Param([Tamir.SharpSsh.SshShell]$SshShell=$global:LastSshSession)
    $SshShell.WriteLine( "exit" )
    sleep -milli 500
    if($SshShell.ShellOpened) { Write-Warning "Shell didn't exit cleanly, closing anyway." }
    $SshShell.Close()
    $SshShell = $null
 }
 
 function Invoke-Ssh {
 Param(
    [string]$command
 ,  [Tamir.SharpSsh.SshShell]$SshShell=$global:LastSshSession
 )
 
    if($SshShell.ShellOpened) {
       $SshShell.WriteLine( $command )
       if($expect) {
          $SshShell.Expect( $expect ).Split("`n")
       }
       else {
          sleep -milli 500
          $SshShell.Expect().Split("`n")
       }
    }
    else { throw "The ssh shell isn't open!" } 
 }
 
 function Send-Ssh {
 Param(
    [string]$command
 ,  [Tamir.SharpSsh.SshShell]$SshShell=$global:LastSshSession
 )
 
    if($SshShell.ShellOpened) {
       $SshShell.WriteLine( $command )
    }
    else { throw "The ssh shell isn't open!" } 
 }
 
 
 function Receive-Ssh {
 Param(
 ,  [Tamir.SharpSsh.SshShell]$SshShell=$global:LastSshSession
 )
    if($SshShell.ShellOpened) {
       if($expect) {
          $SshShell.Expect( $expect ).Split("`n")
       }
       else {
          sleep -milli 500
          $SshShell.Expect().Split("`n")
       }
    }
    else { throw "The ssh shell isn't open!" } 
 }
`

