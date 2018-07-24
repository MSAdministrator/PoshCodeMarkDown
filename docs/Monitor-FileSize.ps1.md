---
Author: paul kiri
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 3576
Published Date: 2013-08-13t06
Archived Date: 2016-04-06t11
---

# monitor-filesize - 

## Description

this function/script monitors the size of a file and all of its sub-directories.  it can also monitor one specific file but the user must specifiy the exact path in that case.  usefull for making sure that disk space on a server doesn’t fill up unexpectedly.

## Comments



## Usage



## TODO



## function

`monitor-filesize`

## Code

`#
 #
 function Monitor-FileSize
 {
 
 <#
 
 .Synopsis
     Checks the file size of a given file until it reaches the specified size
 .Description
     Checks the file size of a given file until it reaches the specified size.  AT that point, it alerts the user as to what the original file-size-boundry was and what it currently is.  The interval at which the script runs can be specified by the user.
 .Parameter FilePath
     Path of the file that will be monitored.  If not pointed to a specific file, the script will montior all sub-directories as well.  ie. if pointed to C:\ drive, will monitor ALL files on C:\ drive
 .Parameter Size
     File size is monitored against this value. When file size is equal or greater than this value, user alert is triggered.  Enter size constraints as integer values (ex: -Size 100 NOT -Size 100kb)
 .Parameter Interval
     The wait interval between the executions of the function. The value of this parameter is in seconds and default value is 5 seconds
 .Example
     Monitor-FileSize -FilePath C:\Test -Size 100
     
     Returns a message to the user alerting them when at least 100kb worth of memory is stored in the selected location.
 .Example
     Monitor-FileSize -FilePath C:\Test -Size 100 -Interval 20
     
     Checking the size of the file and all sub-directories every 20 seconds
 .Notes
     This script cannot be run as a background job and so must have a separate PowerShell window on which it can be running.
 
 	Author: Paul Kiri.
 #>
 
 param
 (
 [Parameter(mandatory=$true,position=0)]
 [string[]]$FilePath
 ,
 [Parameter(mandatory=$true,position=1)]
 [int]$Size
 ,
 [Parameter(mandatory=$false)]
 [int]$Interval=5
 )
 if((Test-Path $FilePath))
 {
    While($FS -le $Size)
    {
       Start-Sleep -Seconds $Interval
       $FileSize = get-childitem $FilePath -Recurse -Include *.* | foreach-object -Process { $_.length / 1024 } | Measure-Object -Sum | Select-Object -Property Sum
       $FS = $FileSize.Sum
    }
 } 
 
 if (Test-Path $FilePath)
 {
 Write-Output "The files at location, $FilePath , have exceeded $Size kb and are now $('{0:N4}' -f $FileSize.Sum) kb in size."
 }
 
 else{"The path '$FilePath' could not be reached or does not exist.  Please check the path name."}
 
 }
`

