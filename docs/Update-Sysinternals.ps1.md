---
Author: sarkisyan
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6509
Published Date: 2016-09-12t07
Archived Date: 2016-09-15t18
---

# update-sysinternals - 

## Description

goo.gl/4sx8n3

## Comments



## Usage



## TODO



## function

`update-sysinternals`

## Code

`#
 #
 function Update-Sysinternals {
   <#
     .SYNOPSIS
         Keeps Sysinternals tools in actual state.
   #>
   param(
     [Parameter(Mandatory=$true)]
     [ValidateNotNullOrEmpty()]
     [ValidateScript({Test-Path $_})]
   )
   
   begin {
     $cur, $new = @{}, @{}
     
     $Path = Convert-Path $Path
     (Get-ChildItem "$Path\*.exe").Where{
       $_.VersionInfo.CompanyName -match 'sysinternals'
     }.ForEach{$cur[$_.Name] = $_.VersionInfo.FileVersion}
     
     Write-Verbose "connecting to Sysinternals..."
     if (!(Test-Path "$(($net = (Get-PSDrive).Where{
       $_.DisplayRoot -match 'sysinternals'
     }).Name):")) {
       Write-Verbose "mount Sysinternals drive..."
       net use * https://live.sysinternals.com | Out-Null
       $net = (Get-PSDrive).Where{$_.DisplayRoot -match 'sysinternals'}
     }
   }
   process {
     Write-Verbose "checking for updates..."
     $cur.Keys.ForEach{
       if ($cur[$_] -ne (
         $$ = (Get-Item "$($net.Name):$_").VersionInfo.FileVersion
       )) { $new[$_] = $$ }
     }
   }
   end {
     if (!$new.Count) {
       Write-Host All tools are already updated. -ForegroundColor green
     }
     else {
       $new.Keys.ForEach{
         if(($p = Get-Process $_.Split('.')[0] -ErrorAction 0)) {
           $p.ForEach{Stop-Process $_.Id -Force}
         }
         Write-Verbose "Update: $_"
         Copy-Item "$($net.Name):$_" $Path -Force
       }
       Write-Host Now all tools has actual version. -ForegroundColor cyan
     }
     Write-Verbose "dismount Sysinternals drive..."
     net use "$($net.Name):" /delete | Out-Null
   }
 }
`

