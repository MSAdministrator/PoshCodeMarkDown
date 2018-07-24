---
Author: lee holmes
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2225
Published Date: 2011-09-09t21
Archived Date: 2017-03-13t02
---

# start-processasuser.ps1 - 

## Description

from windows powershell cookbook (o’reilly) by lee holmes

## Comments



## Usage



## TODO



## function

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
 
 Launch a process under alternate credentials, providing functionality
 similar to runas.exe.
 
 .EXAMPLE
 
 PS >$file = Join-Path ([Environment]::GetFolderPath("System")) certmgr.msc
 PS >Start-ProcessAsUser Administrator mmc $file
 
 #>
 
 param(
     $Credential = (Get-Credential),
 
     [Parameter(Mandatory = $true)]
     [string] $Process,
 
     [string] $ArgumentList = ""
 )
 
 Set-StrictMode -Version Latest
 
 $credential = Get-Credential $credential
 
 if(-not ($credential -is "System.Management.Automation.PsCredential"))
 {
     return
 }
 
 $startInfo = New-Object Diagnostics.ProcessStartInfo
 $startInfo.Filename = $process
 $startInfo.Arguments = $argumentList
 
 if(($credential.Username -eq "$ENV:Username") -or
     ($credential.Username -eq "\$ENV:Username"))
 {
     $startInfo.Verb = "runas"
 }
 else
 {
     $startInfo.UserName = $credential.Username
     $startInfo.Password = $credential.Password
     $startInfo.UseShellExecute = $false
 }
 
 [Diagnostics.Process]::Start($startInfo)
`

