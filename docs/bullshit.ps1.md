---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 4.8.3
Encoding: ascii
License: cc0
PoshCode ID: 3713
Published Date: 
Archived Date: 2012-10-29t05
---

#  - 

## Description

bullshit

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 
 [string]$QTDIR = "C:\Qt\4.8.3-msvc2008"
 [string]$rootDir = "C:\Build\"
 [string]$buildDir = "C:\Build\build_dir\"
 [string]$outputDir = "C:\Build\output_dir\"
 [string]$svnDir = "C:\Build\build_dir\vacuum"
 [string]$svnUrl = "http://vacuum-im.googlecode.com/svn/trunk/"
 [string]$qmake = "qmake.exe"
 
 [Environment]::SetEnvironmentVariable("QTDIR", $QTDIR, "User")
 $env:Path += ";C:\Qt\4.8.3-msvc2008\bin\"
 
 Import-VisualStudioVars -VisualStudioVersion 2008 -Architecture x86
 
 $currentScriptDirectory = split-path -parent $MyInvocation.MyCommand.Definition
 
 [Reflection.Assembly]::LoadFile(($currentScriptDirectory + ".\SharpSvn\SharpSvn.dll"))
 $svnClient = New-Object SharpSvn.SvnClient
 $repoUri = New-Object SharpSvn.SvnUriTarget($svnUrl)
 
 Write-Host "Getting source code from subversion repository" $repoUri -ForegroundColor Green
 $svnClient.CheckOut($repoUri, $svnDir)
 $svnInfo = $null
 $svnClient.GetInfo($SvnDir, ([ref]$svnInfo))
 $rev = $svnInfo.LastChangeRevision
 
 Set-Location $svnDir
 
 Write-Host "Configuring ..." -ForegroundColor Green
 & qmake.exe -r CONFIG-=debug CONFIG+=release INSTALL_PREFIX=C:/Build/output_dir
 
 Write-Host "Building ..." -ForegroundColor Green
 & nmake /NOLOGO
 
 Write-Host "Installing ..." -ForegroundColor Green
 & nmake install /NOLOGO
`

