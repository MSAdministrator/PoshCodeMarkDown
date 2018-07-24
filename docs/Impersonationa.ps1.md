---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 1.1
Encoding: ascii
License: cc0
PoshCode ID: 4480
Published Date: 2015-09-20t10
Archived Date: 2015-12-31t01
---

# impersonationa - 

## Description

a module to solve fileshare permission issues once and for all. allows you to impersonate other network credentials for windows network authentication.

## Comments



## Usage



## TODO



## script

`push-impersonationcontext`

## Code

`#
 #
 
 $MyInvocation.MyCommand.ScriptBlock.Module.OnRemove = { 
    while($script:ImpContextStack.Count) { Pop-ImpersonationContext } 
 }
 
 if(!($Script:UserToysClass =([type]"Huddled.UserToys")))
 {
     $Script:UserToysClass = Add-Type -Namespace Huddled -Name UserToys -MemberDefinition @"
     // http://msdn.microsoft.com/en-us/library/aa378184.aspx
     [DllImport("advapi32.dll", SetLastError = true)]
     public static extern bool LogonUser(string lpszUsername, string lpszDomain, string lpszPassword, int dwLogonType, int dwLogonProvider, ref IntPtr phToken);
 
     // http://msdn.microsoft.com/en-us/library/aa379317.aspx
     [DllImport("advapi32.dll", SetLastError=true)]
     public static extern bool RevertToSelf();
 "@ -passthru
 }
 
 $script:ImpContextStack = new-object System.Collections.Generic.Stack[System.Security.Principal.WindowsImpersonationContext]
 $script:IdStack = new-object System.Collections.Generic.Stack[System.Security.Principal.WindowsIdentity]
 
 function Push-ImpersonationContext {
 <#
 .SYNOPSIS
    Sets the network credentials for the current thread
 .Description
    Stores an identity on the stack and impersonate it for network connections
 .Parameter Credential
    Credentials for authenticating as a new identity.
 .Parameter Name
    A user name for authenticating as a new identity.
 .Parameter Password
    The password (as a String or SecureString) for authenticating as a new identity
 .Parameter Domain
    The domain which goes with the user name for authentication. This is optional, as you can specify a domain or computer name as part of the name using domain\user or user@domain syntax.
 .Parameter Passthru
    Causes Push-ImpersonationContext to output the WindowsIdentity that it's impersonating (not the impersonation context).
 .Example
    Push-ImpersonationContext (Get-Credential)
 .Example
    Push-ImpersonationContext username@domain (Read-Host "Password" -AsSecureString)
 .Example
    $Domain1 = Get-Credential
    $Id1 = PushIC $Domain1 -Passthru
 #>
 [CmdletBinding(DefaultParameterSetName="Credential")]
 Param(
    [Parameter(Position=0,Mandatory=$true,ParameterSetName="Credential")]
    [System.Management.Automation.PSCredential]$Credential
 , 
    [Parameter(Position=0,Mandatory=$true,ParameterSetName="Identity")]
    [Security.Principal.WindowsIdentity]$Identity
 , 
    [Parameter(Position=0,Mandatory=$true,ParameterSetName="Password")]
    [string]$Name
 ,
    [Parameter(Position=1,Mandatory=$true,ParameterSetName="Password")]
    [Alias("PW")]
    $Password = (Read-Host "Password" -AsSecureString)
 ,
    [Parameter(Position=2,Mandatory=$false,ParameterSetName="Password")]
    [string]$Domain
 ,
    [Alias("PT")] 
    [switch]$Passthru
 )
    if(!$Identity) {
       if(!$Credential) {
          if($password -is [string]) {
             $secure = New-Object System.Security.SecureString
             $password.GetEnumerator() | %{ $secure.AppendChar( $_ ) }
             $password = $secure
          }
          if($domain) {
             $user = "${name}@${domain}"
          }
          $Credential = new-object System.Management.Automation.PSCredential $user, $password
       }
 
       Write-Verbose ([Security.Principal.WindowsIdentity]::GetCurrent() | Format-Table Name, Token, User, Groups -Auto | Out-String)
 
       [IntPtr]$userToken = [Security.Principal.WindowsIdentity]::GetCurrent().Token
       if(!$UserToysClass::LogonUser( 
             $Credential.GetNetworkCredential().UserName, 
             $Credential.GetNetworkCredential().Domain, 
             $Credential.GetNetworkCredential().Password, 9, 0, [ref]$userToken)
       ) {
          throw (new-object System.ComponentModel.Win32Exception( [System.Runtime.InteropServices.Marshal]::GetLastWin32Error() ) )
       }
 
       $Identity = New-Object Security.Principal.WindowsIdentity $userToken
    }
    $script:IdStack.Push( $Identity )
    
    $context = $Identity.Impersonate()
    $null = $script:ImpContextStack.Push( $context )
 
    Write-Verbose ([Security.Principal.WindowsIdentity]::GetCurrent() | Format-Table Name, Token, User, Groups -Auto | Out-String)
    
    if($Passthru) { $script:IdStack.Peek() }
 }
 
 function Pop-ImpersonationContext {
 <#
 .Synopsis
    Remove the current impersonation context from the stack and clean it up
 .Description
    Pops the current impersonation context from the stack and undo and dispose it, leaving the former context in place.
 .Param Passthru
    Output the old WindowsIdentity before popping it.
 #>
    param( [switch]$Passthru )
    trap { 
       Write-Error "Impersonation Context Stack is Empty"
       while($script:ImpContextStack.Count -lt $script:IdStack.Count) { $null = $script:IdStack.Pop() }
       return
    }
    if($Passthru) { $script:IdStack.Peek() }
    $context = $script:ImpContextStack.Pop()
    $null = $script:IdStack.Pop()
    
    $context.Undo();
    $context.Dispose();
 }
 
 function Get-ImpersonationContext {
 <#
 .Synopsis
    Display the currently active WindowsIdentity
 #>
    trap { 
       Write-Error "Impersonation Context Stack is Empty"
       return
    }
    Write-Host "There are $($script:ImpContextStack.Count) contexts on the stack"
    while($script:ImpContextStack.Count -lt $script:IdStack.Count) { $null = $script:IdStack.Pop() }
    if($script:ImpContextStack.Count -eq $script:IdStack.Count) {
       $script:IdStack.Peek()
    }
 }
 
 New-Alias popic Pop-ImpersonationContext
 New-Alias pushic Push-ImpersonationContext
 New-Alias gic Get-ImpersonationContext
 Export-ModuleMember -Function * -Alias *
`

