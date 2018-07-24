---
Author: thomas torggler
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 3812
Published Date: 2012-12-04t08
Archived Date: 2012-12-06t03
---

# update-sysinternalssuite - 

## Description

function to download the current sysinternals tools from

## Comments



## Usage



## TODO



## script

`update-sysinternalssuite`

## Code

`#
 #
 <#
 .Synopsis
    Update Sysinternals Suite.
 .DESCRIPTION
    Use PowerShell v3's Invoke-WebRequest do download the latest Sysinternals Tools from: http://live.sysinternals.com. Supports -AsJob
 .EXAMPLE
    Update-SysinternalsSuite -Path C:\tools\sysinterals
    This Example downloads all sysinternals tools to C:\tools\sysinternals.
 .EXAMPLE
    Update-SysinternalsSuite -Path C:\tools\sysinterals -AsJob
    This Example downloads all sysinternals tools to C:\tools\sysinternals, it creates a background job to keep your command line usable!
 #>
 function Update-SysinternalsSuite
 {
     [CmdletBinding(DefaultParameterSetName='Parameter Set 1', 
                   SupportsShouldProcess=$true, 
                   ConfirmImpact='Medium')]
     Param
     (
         [Parameter(Mandatory=$true, 
                    Position=0,
                    ParameterSetName='Parameter Set 1')]
         [ValidateNotNull()]
         [ValidateNotNullOrEmpty()]
         $Path,
 
         [Parameter(Mandatory=$false)]
         [switch] 
         $AsJob
     )
 
     Begin
     {
         if (Test-Path $Path) {
             Write-Verbose "Path exists, updating $Path"
         } else {
             Write-Verbose "Path does not exist, creating folder $path"
             try {
                 New-Item -Path $Path -ItemType Directory -ErrorAction Stop
             } catch {
                 Write-Error "Could not create Folder"
             }
         }
 
         $myscriptblock = {
             param ($path)
             Invoke-WebRequest -Uri live.sysinternals.com | Select-Object -ExpandProperty Links | 
                 ForEach-Object {
                 if ($_.href -like '*.exe' -or $_.href -like '*.chm' -or $_.href -like '*.hlp' -or $_.href -like '*.sys' -or $_.href -like '*.txt' -or $_.href -like '*.cnt'){
                     $str = "http://live.sysinternals.com"+$($_.href)
                     Invoke-WebRequest -Uri $str -OutFile $path$($_.href)
                 }
                 else {
                     Write-Output "Skipped: $($_.href)"
                 }
             }
         
     }
     Process
     {
 
         if ($AsJob) {
             Write-Verbose "AsJob is $AsJob - Creating a background job"
             Start-Job -ScriptBlock $myscriptblock -ArgumentList ($path)
         } else {
             Write-Verbose "AsJob is $AsJob - Running script normally"
             Invoke-Command -ScriptBlock $myscriptblock -ArgumentList ($path)
         }
     }
     End
     {
     }
 }
`

