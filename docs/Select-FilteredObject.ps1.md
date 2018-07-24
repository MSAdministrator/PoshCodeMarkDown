---
Author: lee holmes
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2213
Published Date: 2011-09-09t21
Archived Date: 2016-03-18t21
---

# select-filteredobject.ps - 

## Description

from windows powershell cookbook (o’reilly) by lee holmes

## Comments



## Usage



## TODO



## script

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
 
 Provides an inteactive window to help you select complex sets of objects.
 To do this, it takes all the input from the pipeline, and presents it in a
 notepad window.  Keep any lines that represent objects you want to retain,
 delete the rest, then save the file and exit notepad.
 
 The script then passes the original objects that you kept along the
 pipeline.
 
 .EXAMPLE
 
 Get-Process | Select-FilteredObject | Stop-Process -WhatIf
 Gets all of the processes running on the system, and displays them to you.
 After you've selected the ones you want to stop, it pipes those into the
 Stop-Process cmdlet.
 
 #>
 
 begin
 {
     Set-StrictMode -Version Latest
 
     $filename = [System.IO.Path]::GetTempFileName()
 
     $header = @"
 ############################################################
 ##
 ############################################################
 
 "@
 
     $header > $filename
 
     $objectList = @()
     $counter = 0
 }
 
 process
 {
     "{0}: {1}" -f $counter,$_.ToString() >> $filename
 
     $objectList += $_
     $counter++
 }
 
 end
 {
     $process = Start-Process Notepad -Args $filename -PassThru
     $process.WaitForExit()
 
     foreach($line in (Get-Content $filename))
     {
         if($line -match "^(\d+?):.*")
         {
             $objectList[$matches[1]]
         }
     }
 
     Remove-Item $filename
 }
`

