---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 858
Published Date: 2009-02-10t12
Archived Date: 2016-07-29t11
---

# get-specialpath - 

## Description

this is an enhancement on top of the [environment]

## Comments



## Usage



## TODO



## function

``

## Code

`#
 #
 ###############################################################################
 ##
 #{
    param([string]$folder)
    BEGIN {
       if ($folder.Length -gt 0) { 
          return $folder | &($MyInvocation.InvocationName); 
       } else {
          $WshShellFolders=@{CommonDesktop=0;CommonStartMenu=1;CommonPrograms=2;CommonStartup=3;PrintHood=6;Fonts=8;NetHood=9};
       }
    }
    PROCESS {
       if($_){
          if($_ -eq "QuickLaunch") {
             $f1 = [Environment]::GetFolderPath("ApplicationData")
             return Join-Path $f1 "\Microsoft\Internet Explorer\Quick Launch"
          } elseif($WshShellFolders.Contains($_)){
             if(-not (Test-Path variable:\global:WshShell)) { $global:WshShell = New-Object -com "WScript.Shell" }
             return ([string[]]$global:WshShell.SpecialFolders)[$WshShellFolders[$_]]
          } else {
             trap
             {
                throw new-object system.componentmodel.invalidenumargumentexception $(
                   "Cannot convert value `"$_`" to type `"SpecialFolder`" due to invalid enumeration values. " +
                   "Specify one of the following enumeration values and try again. The possible enumeration values are: " +
                   "Desktop, Programs, Personal, MyDocuments, Favorites, Startup, Recent, SendTo, StartMenu, MyMusic, " +
                   "DesktopDirectory, MyComputer, Templates, ApplicationData, LocalApplicationData, InternetCache, Cookies, " +
                   "History, CommonApplicationData, System, ProgramFiles, MyPictures, CommonProgramFiles, CommonDesktop, " +
                   "CommonStartMenu, CommonPrograms, CommonStartup, PrintHood, Fonts, NetHood, QuickLaunch")
             }
             return [Environment]::GetFolderPath($_)
          }
       }
    }
 #}
`

