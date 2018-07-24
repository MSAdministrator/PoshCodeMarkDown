---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 8.1
Encoding: ascii
License: cc0
PoshCode ID: 6468
Published Date: 
Archived Date: 2016-08-13t22
---

#  - 

## Description

get-dotnetframeworkversion

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
         List installed .Net Framework versions
 
     .DESCRIPTION
         List installed .Net Framework versions.
         Latest version known to this function is .NET Framework 4.6.2 Preview
 
     .EXAMPLE
         Get-DotNetFrameworkVersion
 
     .EXAMPLE
         Get-DotNetFrameworkVersion Server1, Server2
 
     .EXAMPLE
         "Server1", "Server2" | Get-DotNetFrameworkVersion
 
     .OUTPUTS
         [PSObject]                          
 
     .NOTES
         NAME:      Get-DotNetFrameworkVersion
         AUTHOR:    Patrick Sczepanski
         VERSION:   20160804
         
     .LINK
 	https://www.redtoo.com/ch/blog/get-your-net-together/
 
 #>
 param(
     [parameter(ValueFromPipeline = $true, ValueFromPipelineByPropertyName = $true)]
     [string[]]
     $Computer = $env:COMPUTERNAME
 )
 
 begin {
     $dotNetRegistry  = 'SOFTWARE\Microsoft\NET Framework Setup\NDP'
     $dotNet4Registry = 'SOFTWARE\Microsoft\NET Framework Setup\NDP\v4\Full'
     $dotNet4Builds = @{
         30319  = '.NET Framework 4.0'
         378389 = '.NET Framework 4.5'
         378675 = '.NET Framework 4.5.1 (8.1/2012R2)'
         378758 = '.NET Framework 4.5.1 (8/7 SP1/Vista SP2)'
         379893 = '.NET Framework 4.5.2' 
         393295 = '.NET Framework 4.6 (Windows 10)'
         393297 = '.NET Framework 4.6 (NON Windows 10)'
         394254 = '.NET Framework 4.6.1 (Windows 10)'
         394271 = '.NET Framework 4.6.1 (NON Windows 10)'
         394747 = '.NET Framework 4.6.2 Preview (Windows 10 Insider Preview Build 14295)'
         394748 = '.NET Framework 4.6.2 Preview (NON Windows 10)'
     }
     $dotNet4Versions = @{
         30319  = '4.0'
         378389 = '4.5'
         378675 = '4.5.1'
         378758 = '4.5.1'
         379893 = '4.5.2' 
         393295 = '4.6'
         393297 = '4.6'
         394254 = '4.6.1'
         394271 = '4.6.1'
         394747 = '4.6.2'
         394748 = '4.6.2'
     }
 }
 process {
     $Computer |
         Foreach-Object {
 
             if($regKey = [Microsoft.Win32.RegistryKey]::OpenRemoteBaseKey('LocalMachine', $_)) {
 
             if ($netRegKey = $regKey.OpenSubKey("$dotNetRegistry")) {
                 foreach ($versionKeyName in $netRegKey.GetSubKeyNames()) {
                     if ($versionKeyName -match '^v[123]') {
                         $versionKey = $netRegKey.OpenSubKey($versionKeyName)
                         $version = [version]($versionKey.GetValue('Version', ''))
                         New-Object -TypeName PSObject -Property @{
                             ComputerName = $_
                             NetFXBuild = $version.Build
                             NetFXVersion = '.NET Framework ' + $version.Major + '.' + $version.Minor
                             VersionNumber = $version
                         } 
                     }
                 }
             }
 
             if ($net4RegKey = $regKey.OpenSubKey("$dotNet4Registry")) {
                 if(-not ($net4Release = $net4RegKey.GetValue('Release'))) {
                     $net4Release = 30319
                 }
                 New-Object -TypeName PSObject -Property @{
                     ComputerName = $_
                     NetFXBuild = $net4Release
                     NetFXVersion = $dotNet4Builds[$net4Release]
                     VersionNumber = [version]("{0}.{1}" -f $dotNet4Versions[$net4Release], $net4Release)
                 } 
             }
         }
     }
 }
`

