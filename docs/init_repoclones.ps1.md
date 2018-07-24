---
Author: justin dearing
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 3633
Published Date: 2012-09-09t20
Archived Date: 2012-09-15t04
---

# init_repoclones.ps1 - 

## Description

a script for creating clones of one or more svn repositories. it can only clone multiple repositories if they only differ by one substring (e.g. they all are svn repos on google code, sourceforge, codeplex, or some other forge.)

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 <#
 .SYNOPSIS 
 Makes a local copy of one or more remote svn repositories.
 .DESCRIPTION 
 Makes a local copy of one or more remote svn repositories.
 .INPUTS 
 None 
     You cannot pipe objects to init-repoclones.ps1 
 .OUTPUTS
 None
 .EXAMPLE 
 
 #>
 param (
     [string[]] $Repos = @('protobuf', 'mb-unit', 'opennode2'),
     [string] $RemoteRepoFormat = 'http://{0}.googlecode.com/svn/',
     [string] $LocalRepoBasePath = (Split-Path (Get-Variable MyInvocation).Value.MyCommand.Path),
     [switch] $DeleteExisting
 )
 
 @('svnsync.exe', 'svnadmin.exe') | % {
     if (-not (Get-Command $_  -ErrorAction SilentlyContinue)) {
         Write-Host "The executable `"$($_)`" is required by this script and was not found on the path.";
         exit 1;
     }
 }
 
 $Repos | % {
     $localRepoPath = (Join-Path $($LocalRepoBasePath) $_);
     $localRepoUri = ([uri]($localRepoPath)).AbsoluteUri;
     $remoteRepo = [string]::Format($RemoteRepoFormat, $_);
 
     if (Test-Path $localRepoPath) {
         if ($DeleteExisting) {
             Write-Host "$($localRepoPath) already exists. Deleting . . .";
             rm -r -f $localRepoPath;
         }
         else {
             Write-Host "$($localRepoPath) already exists. Skipping initialization . . .";
         }
     }
     else {
         Write-Host "Cloning $($remoteRepo) into $($localRepoPath)"
         & svnadmin create $localRepoPath;
         [IO.File]::WriteAllText((Join-Path $localRepoPath 'hooks\pre-revprop-change.bat'), 'exit 0');
         & svnsync init $localRepoUri $remoteRepo;
     }
     & svnsync sync $localRepoUri $remoteRepo;
 
 }
`

