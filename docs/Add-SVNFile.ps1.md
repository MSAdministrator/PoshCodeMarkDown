---
Author: benduru
Publisher: 
Copyright: 
Email: 
Version: 1.81
Encoding: utf-8
License: cc0
PoshCode ID: 3961
Published Date: 2014-02-18t21
Archived Date: 2014-10-13t03
---

# add-svnfile - 

## Description

upload bucnh of files to svn repository

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 funtcion Add-SVNFile {
     param (
         [Parameter(Position=0,Mandatory=$true,ValueFromPipeline=$true)]
         [String[]]$File,
         [Parameter(Position=1,Mandatory=$true)]
         [Parameter(Position=2,Mandatory=$true)]
         [String]$Comment = "Nothing"
     )
 
     Start-Process -Filepath "svn.exe" -ArgumentList "co $uri" -Wait -WindowStyle Hidden
     Start-Process -Filepath "svn.exe" -ArgumentList "add $File" -Wait -WindowStyle Hidden
     Start-Process -Filepath "svn.exe" -ArgumentList "ci -m $Comment $File" -Wait -WindowStyle Hidden
 
 }
`

