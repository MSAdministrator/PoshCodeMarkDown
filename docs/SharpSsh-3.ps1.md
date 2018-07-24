---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2099
Published Date: 2011-08-24t21
Archived Date: 2017-03-12t05
---

# sharpssh 3 - 

## Description

i’ve tweaked new-sshsession again, this time it supports multiple parameter sets (including one with the password in plain text, and one which takes a pscredential).

## Comments



## Usage



## TODO



## script

`convertto-securestring`

## Code

`#
 #
 
 [void][reflection.assembly]::LoadFrom( (Resolve-Path "~\Documents\WindowsPowerShell\Libraries\Tamir.SharpSSH.dll") )
 
 Function ConvertTo-SecureString {
 #.Synopsis
 Param([string]$input)
    $result = new-object System.Security.SecureString
  
    foreach($c in $input.ToCharArray()) {
       $result.AppendChar($c)
    }
    $result.MakeReadOnly()
    return $result
 }
 
 Function ConvertTo-PSCredential {
 #.Synopsis
 Param([System.Net.NetworkCredential]$Credential)
    $result = new-object System.Security.SecureString
  
    foreach($c in $input.ToCharArray()) {
       $result.AppendChar($c)
    }
    $result.MakeReadOnly()
    New-Object System.Management.Automation.PSCredential "$($Credential.UserName)@$($Credential.Domain)", (ConvertTo-SecureString $Credential.Password)
 }
 
 ##
 
 function New-SshSession {
 <#
 .Synopsis
    Create an SSH session to a remote server
 .Description
    Connect to a specific SSH server with the specified credentials.  Supports RSA KeyFile connections.
 .Parameter HostName
    The server to SSH into
 .Parameter UserName
    The user name to use for login
 .Parameter Password
    The Password for login (Note: you really should pass this as a System.Security.SecureString, but it will accept a string instead)
 .Parameter KeyFile
    An RSA keyfile for ssh secret key authentication (INSTEAD of username/password authentication).
 .Parameter Credentials
    A PSCredential object containing all the information needed to log in. The login name should be in the form user@host
 .Parameter Passthru
    If passthru is specified, the new SshShell is returned.
 .Example 
    New-SshSession Microsoft.com BillG Micr050ft
    
    Description
    -----------
    Creates a new ssh session with the ssh server at Microsoft.com using the 'BillG' as the login name and 'Micr050ft' as the password (please don't bother trying that).
 .Example 
    New-SshSession Microsoft.com -Keyfile BillGates.ppk
    
    Description
    -----------
    Creates a new ssh session with the ssh server at Microsoft.com using the credentials supplied in the BillGates.ppk private key file.
 .Example
    New-SshSession $MSCred
   
    Description
    -----------
    Creates a new ssh session based on the supplied credentials. Uses the output of $MsCred.GetNetworkCredential() for the server to log into (domain) and the username and password.
 #>
 [CmdletBinding(DefaultParameterSetName='NamePass')]
 Param(
    [Parameter(Position=0,Mandatory=$true,ParameterSetName="NamePass",ValueFromPipelineByPropertyName=$true)]
    [Parameter(Position=0,Mandatory=$true,ParameterSetName="RSAKeyFile",ValueFromPipelineByPropertyName=$true)]
    [string]$HostName
 ,
    [Parameter(Position=1,Mandatory=$false,ParameterSetName="NamePass",ValueFromPipelineByPropertyName=$true)]
    [string]$UserName
 ,
    [Parameter(Position=2,Mandatory=$false,ParameterSetName="NamePass",ValueFromPipelineByPropertyName=$true)]
    $Password
 ,  
    [Parameter(Position=1,Mandatory=$true,ParameterSetName="RSAKeyFile",ValueFromPipelineByPropertyName=$true)]
    [string]$KeyFile
 ,
    [Parameter(Position=0,Mandatory=$true,ParameterSetName="PSCredential",ValueFromPipeline=$true)]
    [System.Management.Automation.PSCredential]$Credentials
 ,
    [switch]$Passthru
 )
    process {
       switch($PSCmdlet.ParameterSetName) {
          'RSAKeyFile'   {
             $global:LastSshSession = new-object Tamir.SharpSsh.SshShell $HostName, $UserName
             $global:LastSshSession.AddIdentityFile( (Convert-Path (Resolve-Path $KeyFile)) )
             
          }
          'NamePass' {
             if(!$UserName -or !$Password) {
                $Credentials = $Host.UI.PromptForCredential("SSH Login Credentials",
                                                 "Please specify credentials in user@host format",
                                                 "$UserName@$HostName","")
             } else {
                if($Password -isnot [System.Security.SecureString]) {
                   $Password = ConvertTo-SecureString $Password
                }
                $Credentials = New-Object System.Management.Automation.PSCredential "$UserName@$HostName", $Password
             }
          }
       }
       if($Credentials) {
          $global:LastSshSession = new-object Tamir.SharpSsh.SshShell `
                                           $Credentials.GetNetworkCredential().Domain, 
                                           $Credentials.GetNetworkCredential().UserName,
                                           $Credentials.GetNetworkCredential().Password
       }
 
       $global:LastSshSession.Connect()
       $global:LastSshSession.RemoveTerminalEmulationCharacters = $true
       
       if($Passthru) { return $global:LastSshSession }
       
       $global:LastSshSession.WriteLine("")
       sleep -milli 500
       $global:defaultSshPrompt = [regex]::Escape( $global:LastSshSession.Expect().Split("`n")[-1] )
    }
 }
 
 function Remove-SshSession {
 <#
    .Synopsis
       Exits the open SshSession
 #>
 Param([Tamir.SharpSsh.SshShell]$SshShell=$global:LastSshSession)
    $SshShell.WriteLine( "exit" )
    sleep -milli 500
    if($SshShell.ShellOpened) { Write-Warning "Shell didn't exit cleanly, closing anyway." }
    $SshShell.Close()
    $SshShell = $null
 }
 
 function Invoke-Ssh {
 <#
    .Synopsis
       Executes an SSH command and Receives output
    .Description
       Invoke-Ssh is basically the same as a Send-Ssh followed by a Receive-Ssh, except that Expect defaults to $defaultSshPrompt (which is read in New-SshSession)
    .Parameter Command
       The command to send
    .Parameter Expect
       A regular expression for expect. The ssh session will wait for a line that matches this regular expression to be output before returning, and will return all the text up to AND INCLUDING the line that matches.
       Defaults
    .Parameter SshShell
       The shell to invoke against. Defaults to the LastSshSession
 #>
 Param(
    [string]$Command
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
 <#
    .Synopsis
       Executes an SSH command without receiving input
    .Description
       Sends a command to an ssh session
    .Parameter Command
       The command to send
    .Parameter SshShell
       The shell to send to. Defaults to the LastSshSession
 #>
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
 <#
    .Synopsis
       Receives output from an SSH session
    .Description
       Retrieves output from an SSH session until the text matches the Expect pattern
    .Parameter Expect
       A regular expression for expect. The ssh session will wait for a line that matches this regular expression to be output before returning, and will return all the text up to AND INCLUDING the line that matches.
    .Parameter SshShell
       The shell to wait for. Defaults to the LastSshSession
 #>
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

