---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6645
Published Date: 2016-12-08t21
Archived Date: 2016-12-11t05
---

# [resolvepaths()] - 

## Description

a long time ago, i wrote a resolvepaths attribute which i never really published.

## Comments



## Usage



## TODO



## script

`get-path`

## Code

`#
 #
 using System;
 using System.ComponentModel;
 using System.Management.Automation;
 using System.Collections.ObjectModel;
 using System.Collections.Generic;
 using System.Text.RegularExpressions;
 
 [AttributeUsage(AttributeTargets.Field | AttributeTargets.Property)]
 public class ResolvePathsAttribute : ArgumentTransformationAttribute {
 
    public enum PathType { Simple, Provider, Drive, Relative }
 
    public PathType ResolveAs { get; set; }
 
    public override string ToString() {
       return "[ResolvePaths(" + ((ResolveAs != PathType.Simple) ? "ResolveAs=\"" + ResolveAs + "\")]" : ")]");
    }
 
    public override Object Transform( EngineIntrinsics engine, Object inputData) {
       // standard workaround for the initial bind when pipeline data hasn't arrived
       if(inputData == null) {  return null; }
 
       ProviderInfo provider = null;
       PSDriveInfo drive = null;
       Collection<string> results = new Collection<string>();
       Collection<string> providerPaths = new Collection<string>();
       var PSPath = engine.SessionState.Path;
       var inputPaths = new Collection<string>();
 
       try {
          // in order to not duplicate code, always treat it as an object array
          var inputArray = inputData as object[];
          if(inputArray == null) { inputArray = new object[]{inputData}; }
 
 
          foreach(var input in inputArray) {
             // work around ToString() problem in FileSystemInfo
             var fsi = input as System.IO.FileSystemInfo;
             if(fsi != null) {
                inputPaths.Add(fsi.FullName);
             } else {
                // work around FileSystemInfo actually being a PSObject
                var psO = input as System.Management.Automation.PSObject;
                if(psO != null) {
                   fsi = psO.BaseObject as System.IO.FileSystemInfo;
                   if(fsi != null) {
                      inputPaths.Add(fsi.FullName);
                   } else {
                      inputPaths.Add(psO.BaseObject.ToString());
                   }
                } else {
                   inputPaths.Add(input.ToString());
                }
             }
          }
 
          foreach(string inputPath in inputPaths) {
 
             if(WildcardPattern.ContainsWildcardCharacters(inputPath)) {
                providerPaths = PSPath.GetResolvedProviderPathFromPSPath(inputPath, out provider);
             } else {
                providerPaths.Add(PSPath.GetUnresolvedProviderPathFromPSPath(inputPath, out provider, out drive));
             }
 
             foreach(string path in providerPaths) {
                var newPath = path;
 
                if(ResolveAs == PathType.Provider && !PSPath.IsProviderQualified(newPath)) {
                   newPath = provider.Name + "::" + newPath;
                } 
                else if(ResolveAs != PathType.Provider && PSPath.IsProviderQualified(newPath)) 
                {
                   newPath = Regex.Replace( newPath, Regex.Escape( provider.Name + "::" ), "");
                }
 
                if(ResolveAs == PathType.Drive) 
                {
                   string driveName;
                   if(!PSPath.IsPSAbsolute(newPath, out driveName)) {
                      if(drive == null) {
                         newPath = PSPath.GetUnresolvedProviderPathFromPSPath(newPath, out provider, out drive);
                      }
                      if(!PSPath.IsPSAbsolute(newPath, out driveName)) {
                         newPath = drive.Name + ":\\" + PSPath.NormalizeRelativePath( newPath, drive.Root );
                      }
                   }
                } else if(ResolveAs == PathType.Relative) {
                   var currentPath = PSPath.CurrentProviderLocation(provider.Name).ProviderPath.TrimEnd(new[]{'\\','/'});
                   var relativePath = Regex.Replace(newPath, "^" + Regex.Escape(currentPath), "", RegexOptions.IgnoreCase);
                   // Console.WriteLine("currentPath: " + currentPath + "  || relativePath: " + relativePath);
                   if(relativePath != newPath) {
                      newPath = ".\\" + relativePath.TrimStart(new[]{'\\'});
                   } else {
                      try {
                         newPath = PSPath.NormalizeRelativePath(newPath, currentPath);
                         // Console.WriteLine("currentPath: " + currentPath + "  || relativePath: " + relativePath + "  || newPath: " + newPath);
                      } catch {
                         newPath = relativePath;
                      }
                   }
                }
 
                results.Add(newPath);
             }
          }
       } catch (ArgumentTransformationMetadataException) {
          throw;
       } catch (Exception e) {
          throw new ArgumentTransformationMetadataException(string.Format("Cannot determine path ('{0}'). See $Error[0].Exception.InnerException.InnerException for more details.",e.Message), e);
       }
       return results;
    }
 }
 '@ 
 
 ##############################
 
    function Get-Path {
       param(
          [Parameter(Position=0,Mandatory=$true,ValueFromPipeline=$true,ValueFromPipelineByPropertyName=$true)]
          [Alias("PSPath")][String[]]
          [ResolvePaths()]
          $Path
       )
       process { $Path }
    }
 
    function Get-DrivePath {
       param(
          [Parameter(Mandatory=$true,ParameterSetName="Resolved",Position=0,ValueFromPipeline=$true,ValueFromPipelineByPropertyName=$true)]
          [String[]][ResolvePaths(ResolveAs="Drive")]
          [Alias("PSPath")]$Path
       )
       process { $Path }
    }
 
    function Get-ProviderPath {
       param(
          [Parameter(Mandatory=$true,ParameterSetName="Resolved",Position=0,ValueFromPipeline=$true,ValueFromPipelineByPropertyName=$true)]
          [String[]][ResolvePaths(ResolveAs="Provider")]
          [Alias("PSPath")]$Path
       )
       process { $Path }
    }
 
    function Get-RelativePath {
       param(
          [Parameter(Mandatory=$true,ParameterSetName="Resolved",Position=0,ValueFromPipeline=$true,ValueFromPipelineByPropertyName=$true)]
          [String[]][ResolvePaths(ResolveAs="Relative")]
          [Alias("PSPath")]$Path
       )
       process { $Path }
    }
 
 
    function Copy-WhatIf {
       param(
          [Parameter(Position=0,Mandatory=$true,ValueFromPipeline=$true,ValueFromPipelineByPropertyName=$true)]
          [Alias("PSPath")][String[]]
          [ResolvePaths()]
          $Source,
 
          [Parameter(Position=1,Mandatory=$true)]
          [String]
          [ResolvePaths()]
          $Destination
       )
       process { 
          if(!(Test-Path $Destination)) {
             mkdir $Destination -whatif
          }
          Copy-Item $Source $Destination -whatif
       }
    }   
 
 
 #########################
 
    function Assert-Equal {
       Param($expected,[scriptblock]$code,$errorMessage)
       $output = &$code
       if( $expected -is [scriptblock]) {
          $expected = &$expected
       }
       if( $Expected -is [Array]) {
          if( compare-object $expected $output ) {
             Write-Warning ("Expected '$expected', but got '$output'`n+`t" + $MyInvocation.PositionMessage)
          }
       } else {
          if( $expected -ne $output ) {
             Write-Warning ("Expected '$expected', but got '$output'`n+`t" + $MyInvocation.PositionMessage)
          }
       }
    }
 
    Push-Location C:\users\ | out-null
 
 
    cd  C:\users\ | out-null
 
    Assert-Equal "C:\NotAUser"       { get-path C:\NotAUser }
    Assert-Equal "C:\users\NotAUser" { get-path C:\users\NotAUser }
    Assert-Equal "C:\users\NotAUser" { get-path C:NotAUser }
    Assert-Equal "C:\users\NotAUser" { get-path NotAUser }
 
    Assert-Equal "FileSystem::C:\NotAUser"       { Get-ProviderPath C:\NotAUser }
    Assert-Equal "FileSystem::C:\users\NotAUser" { Get-ProviderPath C:\users\NotAUser }
    Assert-Equal "FileSystem::C:\users\NotAUser" { Get-ProviderPath C:NotAUser }
    Assert-Equal "FileSystem::C:\users\NotAUser" { Get-ProviderPath NotAUser }
 
    Assert-Equal "..\NotAUser"       { Get-RelativePath C:\NotAUser }
    Assert-Equal ".\NotAUser" { Get-RelativePath C:\users\NotAUser }
    Assert-Equal ".\NotAUser" { Get-RelativePath C:NotAUser }
    Assert-Equal ".\NotAUser" { Get-RelativePath NotAUser }
 
    Assert-Equal { convert-path Public } { get-path Public }
    Assert-Equal { convert-path C:\users\Public } { get-path C:\users\Public }
    Assert-Equal { convert-path C:Public } { get-path C:Public }
    Assert-Equal { convert-path Public\* } { get-path Public\* }
    Assert-Equal { convert-path C:\*\Public } { get-path C:\*\Public }
 
    Assert-Equal { Resolve-Path Public -Relative } { Get-RelativePath Public }
    Assert-Equal { Resolve-Path C:\users\Public -Relative } { Get-RelativePath C:\users\Public }
    Assert-Equal { Resolve-Path C:Public -Relative } { Get-RelativePath C:Public }
    Assert-Equal { Resolve-Path Public\* -Relative } { Get-RelativePath Public\* }
    Assert-Equal { Resolve-Path C:\*\Public -Relative } { Get-RelativePath C:\*\Public }
 
    Assert-Equal { convert-path C:*\Documents } { get-path C:*\Documents }   
    Assert-Equal { convert-path C:\Users\*\Documents, C:\Users\*\Desktop } { get-path C:\Users\*\Documents, C:\Users\*\Desktop }
 
 
    cd hklm:\software\microsoft | out-null
 
    Assert-Equal "FileSystem::C:\Test"       { Get-ProviderPath C:\Test }
    Assert-Equal "FileSystem::C:\users\Test" { Get-ProviderPath C:\users\Test }
    Assert-Equal "FileSystem::C:\users\Test" { Get-ProviderPath C:Test }
 
    Assert-Equal "HKEY_LOCAL_MACHINE\software\microsoft" { get-path hklm:\software\microsoft }
    Assert-Equal "HKEY_LOCAL_MACHINE\software\microsoft\Windows" { get-path Windows }
    Assert-Equal "HKEY_LOCAL_MACHINE\software\microsoft\Windows" { get-path hklm:Windows }
 
 
    Assert-Equal ".\" { Get-RelativePath hklm:\software\microsoft }
    Assert-Equal ".\Windows" { Get-RelativePath Windows }
    Assert-Equal ".\Windows" { Get-RelativePath hklm:Windows }
    Assert-Equal "..\Classes" { Get-RelativePath hklm:\SOFTWARE\Classes }
    Assert-Equal "..\..\SYSTEM\CurrentControlSet" { Get-RelativePath hklm:\SYSTEM\CurrentControlSet }
    Assert-Equal "HKEY_CURRENT_USER\Network" { Get-RelativePath hkcu:\Network }
 
    Assert-Equal { convert-path Windows } { get-path Windows }
    Assert-Equal { convert-path hklm:\software\microsoft } { get-path hklm:\software\microsoft }
    Assert-Equal { convert-path hklm:Windows } { get-path hklm:Windows }
 
    Assert-Equal "Registry::HKEY_LOCAL_MACHINE\software\microsoft\Test" { Get-ProviderPath Test }
 
    cd variable: | out-null
    Assert-Equal "Variable::Test" { Get-ProviderPath Test }
    Assert-Equal "Environment::Test" { Get-ProviderPath env:\Test }
    cd env: | out-null
    Assert-Equal "Variable::Test" { Get-ProviderPath variable:Test }
    Assert-Equal "Environment::Test" { Get-ProviderPath Test }
 
    cd hklm:\software\microsoft | out-null
    Assert-Equal "HKEY_LOCAL_MACHINE\software\microsoft\Test" { get-path Test }
    Assert-Equal "hklm:\software\microsoft\Test" { Get-DrivePath Test }
    Assert-Equal "Registry::HKEY_LOCAL_MACHINE\software\microsoft\Test" { Get-ProviderPath Test}
    cd variable: | out-null
    Assert-Equal "Variable:\Test" { Get-DrivePath Test }
    Assert-Equal "Env:\Test" { Get-DrivePath env:\Test }
    cd env: | out-null
    Assert-Equal "Variable:\Test" { Get-DrivePath variable:Test }
    Assert-Equal "Env:\Test" { Get-DrivePath Test }
 
    cd c:
    if(test-path ~\Documents\WindowsPowerShell\TestData) {
       cd ~\Documents\WindowsPowerShell\TestData | out-null
 
       Assert-Equal {ls -s | convert-path } {ls -s | Get-Path}
       Assert-Equal {ls -s | resolve-path -relative } {ls -s | Get-RelativePath}
 
    }
 
    Pop-Location
`

